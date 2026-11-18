# Chapter 9: External Objects and IO (manual pages 269-308)

<!-- manual page 269 -->
Chapter 9
External Objects and IO
We use the object-message system to allow interactions with external objects that represent
external entities with state. This section explains Maude’s support for rewriting with external
objects and an implementation of sockets, standard streams, files, processes, real time, and
meta-interpreters as the first such external objects.
Configurations that want to communicate with external objects must contain at least one
portal, where
sort Portal .
subsort Portal < Configuration .
op <> : -> Portal [ctor portal] .
is part of the predefined module CONFIGURATION in the file prelude.maude. Only configura-
tions having a portal are allowed to exchange messages with external objects. This means that
a message from an external object will only be delivered to a configuration that contains an
object with the target Oid and a portal. Configurations are ephemeral—they can be split and
merged by arbitrary rewriting and they do not have an identifier—so this rule is needed to
determine where a message from an external object should be delivered.
Note that although there is a restriction forbidding having multiple objects with the same
identifier in the same configuration, there might be objects with the same identifier in different
configurations. This means that as long as only one such configuration has a portal, messages
from external objects will not be misdelivered. This requirement may arise, for example, with
nested configurations where the outer configuration manages communication between external
objects and objects in the inner configuration and objects in both configurations share identi-
fiers. In this case, only the outer configuration should have a portal, being the only one able to
communicate directly with external objects. What makes the constant <> a portal is its portal
attribute.
Rewriting with external objects is started by the external rewrite command erewrite (ab-
breviated erew) which is like frewrite (see Sections 5.4 and 6.3) except that it allows messages
to be exchanged with external objects that do not reside in the configuration. A few clarifica-
tions on its behavior::
1. Maude will check for external events after every fair traversal of the term.
2. An external event may result in a message injected into a configuration that enables more
internal rewrites.
3. When there are no internal rewrites available but there are requests pending on external
objects (such as reading on a socket or waiting for a process to exit), rather than finishing
and returning to the command line, Maude will suspend on external events. If Maude
257

<!-- manual page 270 -->
258 CHAPTER 9. EXTERNAL OBJECTS AND IO
returned to the command line when there were still external transactions pending, but
no internal rewrites possible, these transactions would be abruptly terminated and any
Maude program with external transactions that depended on the environment would con-
tain a race between the internal rewriting and the environment leading to unpredictable
results.
4. Rewrites that involve messages entering or leaving the configuration do not show up in
tracing, profiling, or rewrite counts.
Note that, even if there are no more rewrites possible, erewrite may not respond; if there
are requests made to external objects that have not yet been fulfilled because of waiting for
external events from the operating system, the Maude interpreter will suspend until at least one
of those events occurs, at which time rewriting will resume. While the interpreter is suspended,
the command erewrite may be aborted with ^C. External objects created by an erewrite
command do not survive to the next erewrite. If a message to an external object is ill-formed
or inappropriate, or the external object is not ready for it, it just stays in the configuration for
future acceptance or for debugging purposes.
Certain predefined external objects are available and some of them are object managers that
can create ephemeral external objects that represent entities such as files and sockets, and, as
we will see in Chapter 19, virtual copies of the Maude interpreter itself.
Access to external objects represents a potential threat. It is difficult for a user to tell,
just by looking at it, if an arbitrary Maude program contains malware. Furthermore, a Maude
program can assemble meta-malware using innocent looking code and then execute it in a
meta-interpreter. File handling and process execution are now disabled by default. These
features need to be enabled with the command line flags -allow-files , -allow-processes,
and -allow-dir. To run completely trusted code, the command line flag -trust enables all
dangerous features. See Appendix A.1.
9.1 Standard streams
Each Unix process has three I/O channels, called standard streams: standard input (stdin),
standard output (stdout), and standard error (stderr). In Maude, these are represented
as three unique external objects, that are defined in a predefined module STD-STREAM in the
file.maude file included in the Maude distribution. Because some of the messages that are
useful for streams are also useful for file I/O, these messages are pulled out into a module
COMMON-MESSAGES.
mod COMMON-MESSAGES is
protecting STRING .
including CONFIGURATION .
op gotLine : Oid Oid String -> Msg [ctor msg format (m o)] .
op write : Oid Oid String -> Msg [ctor msg format (b o)] .
op wrote : Oid Oid -> Msg [ctor msg format (m o)] .
endm
mod STD-STREAM is
including COMMON-MESSAGES .
op getLine : Oid Oid String -> Msg [ctor msg ...] .
op cancelGetLine : Oid Oid -> Msg [ctor msg ...] .
op canceledGetLine : Oid Oid -> Msg [ctor msg ...] .
op streamError : Oid Oid String -> Msg [ctor msg ...] .

<!-- manual page 271 -->
9.1. STANDARD STREAMS 259
op stdin : -> Oid [special (...)] .
op stdout : -> Oid [special (...)] .
op stderr : -> Oid [special (...)] .
endm
I/O on standard streams is always line-oriented and in text mode. stdout and stderr
accept the write() message just like file handlers. They are automatically flushed so this is
the only message they can accept. They always return a wrote() message.
The stdin object accepts a three-argument getLine() message:
getLine(stdin, ME, PROMPT)
ME is the identifier of the sender object, to which the answer will be sent. PROMPT is a string that
will be shown to inform the user that some input is expected. Once the message is processed,
when the user hits the return key, stdin replies with a message
gotLine(ME, stdin, TEXT)
where TEXT contains just a "\n" character in the case of an empty line, and is empty in the
case of an error or end-of-file (^D).
A getLine message will result in a gotLine message with the string corresponding to a line
of text entered in the standard input channel. However, lines ended with a backslash character
and a newline character will be continued. In other words, a backslash-newline combination
indicates that the input line is to be continued, allowing the user to enter long texts, in multiple
lines; see Section 9.1.1 for an example.
The getLine operation is non-blocking on stdin — other computations can take place
while Maude is waiting for the user’s response — and it can be canceled with
cancelGetLine(stdin, ME)
If this message is processed before the line is actually read, it will be replied with
canceledGetLine(ME, stdin)
If a canceledGetLine is sent with no getLine in progress, the cancelGetLine message is
quietly discarded. The following example illustrates this behavior.
omod CANCEL-GET-LINE is
inc STD-STREAM .
pr NAT .
class MyClass | state : Nat .
op myObj : -> Oid .
op run : -> Configuration .
eq run = <> < myObj : MyClass | state : 0 > .
rl < myObj : MyClass | state : 0 >
=> < myObj : MyClass | state : 1 >
getLine(stdin, myObj, "What is your name? ") .
rl < myObj : MyClass | state : 1 >
=> < myObj : MyClass | state : 2 >
cancelGetLine(stdin, myObj) .
endom
Maude> erew run .
erewrite in CANCEL-GET-LINE : run .
rewrites: 3 in 4ms cpu (5ms real) (645 rewrites/second)
result Configuration: <> canceledGetLine(myObj, stdin) < myObj : MyClass | state : 2 >

<!-- manual page 272 -->
260 CHAPTER 9. EXTERNAL OBJECTS AND IO
Typing control-C (interrupt) while waiting for a getLine to be consumed returns the empty
string, as does control-D (EOF), though the former also interrupts the parent process and may
enter the debugger or print a message about being suspended, depending on whether the parent
process is running (doing rewrites) or suspended (awaiting external events) at that instant.
Errors while reading and writing on streams are reported with the
streamError(ME, STREAM, CAUSE)
where CAUSE is a string describing the error cause. For example, an attempt to write an empty
stringtothestandardoutputwithwrite(stdout, ME, "")isrepliedwithastreamError(ME,
stdout, "Empty string.") message.
9.1.1 The Hello Word! example
The module HELLO below shows a very simple program implementing an interaction with the
user, which is asked to introduce his/her name to be properly greeted. The equation for run
produces a starting configuration, containing the portal, a user object to receive messages, and
a message to stdin to read a line of text from the keyboard. When stdin has a line of text,
it sends the text to the requesting object in a gotLine message.
omod HELLO is
inc STD-STREAM .
pr NAT .
class MyClass | state : Nat .
op myObj : -> Oid .
op run : -> Configuration .
var O : Oid .
var S : String .
eq run = <> < myObj : MyClass | state : 0 > .
rl < myObj : MyClass | state : 0 >
=> < myObj : MyClass | state : 1 >
getLine(stdin, myObj, "What is your name? ") .
rl < myObj : MyClass | >
gotLine(myObj, O, S)
=> < myObj : MyClass | >
if S =/= ""
then write(stdout, myObj, "Hello " + S)
else none
fi .
endom
Maude> erew run .
What is your name? Zaphod Beeblebrox
Hello Zaphod Beeblebrox
result Configuration: <> wrote(myObj, stdout) < myObj : MyClass | state : 1 >
As already pointed out, even though standard streams are line-oriented, when handling
a getLine message, a backslash-newline combination indicates that the input line is to be
continued. The backslash-newline is deleted and the continuation lines are prompted; for
example:
Maude> erew run .
erewrite in HELLO : run .
What is your name? Zaphod \
> Beeblebrox

<!-- manual page 273 -->
9.1. STANDARD STREAMS 261
Hello Zaphod Beeblebrox
result Configuration: <> wrote(myObj, stdout) < myObj : MyClass | state : 1 >
9.1.2 A ROT13 cypher example
As a second example of the use of standard streams, let us consider the specification in the
module ROT13 below. The example prints out a banner and then reads lines from the keyboard,
encrypts each line using the ROT13 cypher, and finally prints out the result.
mod ROT13 is
including STD-STREAM .
protecting INT .
op Class : -> Cid .
op myObj : -> Oid .
op run : -> Configuration .
op rot13 : String -> String .
vars O O2 : Oid .
var A : AttributeSet .
vars S T : String .
var C : Char .
eq rot13(C)
= if C >= "A" and C <= "Z"
then char(ascii("A") + ((13 + (ascii(C) - ascii("A"))) rem 26))
else if C >= "a" and C <= "z"
then char(ascii("a") + ((13 + (ascii(C) - ascii("a"))) rem 26))
else C
fi
fi .
eq rot13(S)
[owise] .
= rot13(substr(S, 0, 1)) + rot13(substr(S, 1, length(S)))
eq run
= <>
< myObj : Class | none >
write(stdout, myObj, "\nROT13 Encryption\n----------------\n") .
rl < O : Class | A >
wrote(O, O2)
=> < O : Class | A >
getLine(stdin, O, "Enter plain text> ") .
rl < O : Class | A >
gotLine(O, O2, T)
=> < O : Class | A >
if T =/= ""
then write(stdout, O, "Cypher text: " + rot13(T) + "\n")
else none
fi .
endm

<!-- manual page 274 -->
262 CHAPTER 9. EXTERNAL OBJECTS AND IO
The equation for run produces a starting configuration, containing the portal, a user object
to receive messages, and an initial message to stdout to print out the banner. Each time
stdout prints out some text, it sends a wrote reply to the requesting object; this is matched
in the first rule which then sends a getLine message to stdin to read a line of text from the
keyboard. When stdinhas a line of text, it sends the text to the requesting object in a gotLine
message; the second rule matches this message, calls the equationally defined rot13 function to
encrypt the text, and sends a message to stdout to print the encrypted text. When the wrote
reply comes back, the sequence repeats.
We can have the following interaction:
Maude> erewrite run .
erewrite in ROT13 : run .
ROT13 Encryption
----------------
Enter plain text> Maude
Cypher text: Znhqr
Enter plain text>
rewrites: 107 in 0ms cpu (3364ms real) (149025 rewrites/second)
result Configuration: <> < myObj : Class | none >
9.1.3 A calculator example
ThefollowingprogramillustratestheuseoffunctionsmetaReduceandmetaParse(frommodule
META-LEVEL, described later in Chapter 17) and functions tokenize and printTokens (from
module LEXICAL in Section 8.12) to read Maude arithmetic expressions from the terminal and
evaluate them.
mod CALCULATOR is
inc STD-STREAM .
pr LEXICAL .
pr META-LEVEL .
op myClass : -> Cid .
op myObj : -> Oid .
op run : -> Configuration .
vars O O2 : Oid .
var A : AttributeSet .
vars S T : String .
op compute : String -> String .
eq compute(S) = compute2(metaParse([’CONVERSION], tokenize(S), anyType)) .
op compute2 : ResultPair? -> String .
eq compute2({T:Term, Q:Qid}) = compute3(metaReduce([’CONVERSION], T:Term)) .
eq compute2(noParse(N:Nat)) = printTokens(’\r) + "syntax error" + printTokens(’\o) .
op compute3 : ResultPair -> String .
eq compute3({T:Term, Q:Qid})
= if getKind([’CONVERSION], Q:Qid) == Q:Qid
then printTokens(’\r) + "wrong command" + printTokens(’\o)
else printTokens(metaPrettyPrint([’CONVERSION], T:Term))

<!-- manual page 275 -->
9.2. FILE I/O 263
fi .
eq run
= <>
< myObj : myClass | none >
write(stdout, myObj, "\nCalculator\n------------\n(enter ’exit’ to terminate)\n") .
rl < O : myClass | A >
wrote(O, O2)
=> < O : myClass | A >
getLine(stdin, O, "Expression> ") .
rl < O : myClass | A >
gotLine(O, O2, T)
=> < O : myClass | A >
if T == "exit\n"
then none
else write(stdout, O, "Answer: " + compute(T) + "\n")
fi .
endm
Inthisspecification, anobjectofclassmyClassinteractswiththestdinandstdoutexternal
objects to read and write messages from and to the user. If the text received from stdin is
"exit\n", then the interaction terminates. Otherwise, the compute, compute2 and compute3
operations handle the received string. Since parsing takes place in the CONVERSION predefined
module, the answer will be either a term of sort ResultPair or noParse(n ), in which case a
syntax error message is given. Note that the metaReduce operation may still result in a term
in a kind. If this is the case, a wrong command message is given. Otherwise the pretty printing
of the obtained result is provided.
Then we can execute, for example:
Maude> erew run .
erewrite in CALCULATOR : run .
Calculator
------------
(enter ’exit’ to terminate)
Expression> 3/6 + 12/32
Answer: 7/8
Expression> foo
Answer: syntax error
Expression> 1 / 0
Answer: wrong command
Expression> exit
rewrites: 74 in 10ms cpu (27389ms real) (7247 rewrites/second)
result Configuration: <> < myObj : myClass | none >
Maude>
9.2 File I/O
As pointed out in the introduction of the chapter, access to external objects represents a
potential threat. File handling is disabled by default. The feature may be enabled with the
command line flags -allow-files or -trust. See Appendix A.1.
Unlike standard streams, of which there are exactly three, a Unix process may have many
different files open at any one time. Thus, in order to create new file handle objects as needed,

<!-- manual page 276 -->
264 CHAPTER 9. EXTERNAL OBJECTS AND IO
we have a unique external object called fileManager. To open a file, the fileManager is sent
a message openFile. On success, an openedFile message is returned, with the name of an
external object that is a handle on the open file as one of its arguments. Messages to read and
write the file can be directed to the handle object. On failure, a fileErrormessage is returned,
with a text explanation of why the file could not be opened as one of its arguments. These
messages are defined in the module FILE, which is distributed as part of the Maude system in
the file.maude file.
mod FILE is
including COMMON-MESSAGES .
protecting INT .
sorts Base LinkType .
ops start current end : -> Base [ctor] .
ops hard symbolic : -> LinkType [ctor] .
op file : Nat -> Oid [ctor] .
op openFile : Oid Oid String String -> Msg [ctor msg format (b o)] .
op openedFile : Oid Oid Oid -> Msg [ctor msg format (m o)] .
op removeFile : Oid Oid String -> Msg [ctor msg format (b o)] .
op removedFile : Oid Oid -> Msg [ctor msg format (m o)] .
op getLine : Oid Oid -> Msg [ctor msg format (b o)] .
op getChars : Oid Oid Nat -> Msg [ctor msg format (b o)] .
op gotChars : Oid Oid String -> Msg [ctor msg format (m o)] .
op flush : Oid Oid -> Msg [ctor msg format (b o)] .
op flushed : Oid Oid -> Msg [ctor msg format (m o)] .
op setPosition : Oid Oid Int Base -> Msg [ctor msg format (b o)] .
op positionSet : Oid Oid -> Msg [ctor msg format (m o)] .
op getPosition : Oid Oid -> Msg [ctor msg format (b o)] .
op positionGot : Oid Oid Nat -> Msg [ctor msg format (m o)] .
op closeFile : Oid Oid -> Msg [ctor msg format (b o)] .
op closedFile : Oid Oid -> Msg [ctor msg format (m o)] .
op makeLink : Oid Oid String String LinkType -> Msg [ctor msg format (b o)] .
op madeLink : Oid Oid -> Msg [ctor msg format (m o)] .
op fileError : Oid Oid String -> Msg [ctor msg format (m o)] .
op fileManager : -> Oid [special (...)] .
endm
ThisAPIbasicallywrapstheCstdio library. Correctlyaddressedmessageswithappropriate
message constructors are always accepted. If there is a problem with other message arguments
a fileError reply is generated. Message strings include "Bad mode.", "Bad file name.",
"File operations disabled.", "Bad characters.", "File not open for writing.", etc.
Message strings that do not end in a period, such as "No such file or directory", are
generated by the operating system. Note that these message strings are intended as information
for the user, rather than to be used programmatically.

<!-- manual page 277 -->
9.2. FILE I/O 265
To open a file you send fileManager a message
openFile(fileManager, ME, PATH, MODE)
where ME is the name of the object the reply should be sent to, PATH is the path of the file you
want to open, and MODE is one of: r, r+, w, w+, a, a+.
1
The reply is either
openedFile(ME, fileManager, FILE-HANDLE)
or
fileError(ME, fileManager, REASON)
where FILE-HANDLE is the name of the object used to access the file, and REASON is a textual
explanation from the operating system of what went wrong.
If the file was opened for writing you can send
write(FILE-HANDLE, ME, DATA)
to write data to the file, and receive a reply
wrote(ME, FILE-HANDLE)
or
fileError(ME, FILE-HANDLE, REASON)
A file that is opened for writing can also be sent
flush(FILE-HANDLE, ME)
to flush any buffered data and receive a reply
flushed(ME, FILE-HANDLE)
or
fileError(ME, FILE-HANDLE, REASON)
A file that is opened for reading can be read on a line-by-line basis using the message:
getLine(FILE-HANDLE, ME)
where the reply is either
gotLine(ME, FILE-HANDLE, TEXT)
or
fileError(ME, FILE-HANDLE, REASON)
1The opening modes are exactly the same as those for the C standard library function fopen(). The following
text in provided by the BSD fopen manpage (see https://www.freebsd.org/):
The argument mode points to a string beginning with one of the following letters:
“r" Open for reading. The stream is positioned at the beginning of the file. Fail if the file does
not exist.
“w" Open for writing. The stream is positioned at the beginning of the file. Truncate the file to
zero length if it exists or create the file if it does not exist.
“a" Open for writing. The stream is positioned at the end of the file. Subsequent writes to the
file will always end up at the then current end of file [...]. Create the file if it does not exist.
An optional “+" following “r", “w", or “a" opens the file for both reading and writing.

<!-- manual page 278 -->
266 CHAPTER 9. EXTERNAL OBJECTS AND IO
Here TEXT includes the newline character (if present, since end-of-file also ends the current
line). TEXT is empty to indicate end-of-file with no more characters to read.
A file that is opened for reading can also be read on a character basis (which is more
appropriate for binary files) using the message
getChars(FILE-HANDLE, ME, #CHARS-TO-GET)
where the reply is either
gotChars(ME, FILE-HANDLE, DATA)
or
fileError(ME, FILE-HANDLE, REASON)
Here if DATA is shorter than requested, it indicates the end-of-file was reached.
Reading and writing share a common position into the file where the next read or write
takes place. This position in bytes from the start of the file can be obtained by sending the
message
getPosition(FILE-HANDLE, ME)
where the reply is either
positionGot(ME, FILE-HANDLE, OFFSET)
or
fileError(ME, FILE-HANDLE, REASON)
The position may be changed with the message
setPosition(FILE-HANDLE, ME, OFFSET, BASE)
Here the OFFSET is relative to BASE, where BASE can take one of three values: start, the start
of the file, current, the current position, or end, the end of the file. In the current and end
cases, negative values of OFFSET are allowed. The reply is either
positionSet(ME, FILE-HANDLE)
or
fileError(ME, FILE-HANDLE, REASON)
Finally, an open file can be closed with the message
closeFile(FILE-HANDLE, ME)
Since it is always OK to close an open file, the response is always
closedFile(ME, FILE-HANDLE)
Note that messages that are not recognized or that are sent to nonexistent objects will
be silently ignored and left in the configuration. Messages that are recognized but are not
appropriate for the object they are sent to or which have bad arguments will similarly be
ignored but will generate a “message declined” advisory.
In addition to the previous messages for operating with the contents of files, some filesystem
operations are also available in the API. Files can be removed with the removeFile message
removeFile(fileManager, ME, PATH)
whose reply is
removedFile(ME, fileManager)

<!-- manual page 279 -->
9.2. FILE I/O 267
or a fileError message if an error occurs. Links between files can be created with
makeLink(fileManager, ME, TARGET-PATH, LINK-PATH, TYPE)
that creates a new file in LINK-PATH pointing to TARGET-PATH, where TYPE can be either
symbolic for a symbolic link or hard for a hard link. The message is answered with
madeLink(ME, fileManager)
or with a fileError in case of error.
9.2.1 A file copy example
The COPY-FILE module below illustrates the basic use of files. The module imports the MAYBE
module in Section 7.4.3 instantiated with the Oid view. It specifies a simple algorithm to copy
files. In this case, the run operator takes two arguments, namely the name of the file to be
copied and the name of the new file. The equation for run produces a starting configuration,
containing the portal, a user object to receive messages, and an initial message to open the
original file. Once it is opened, the new file is created. Notice the "w"argument of the openFile
message. Once both files are opened, a loop in which a line is read from the original file and
written in the copy file is initiated. This loop ends when the end of the file is reached. Both
files are then closed.
view Oid from TRIV to CONFIGURATION is
sort Elt to Oid .
endv
mod COPY-FILE is
inc FILE .
pr MAYBE{Oid} * (op maybe{Oid} to null) .
op myClass : -> Cid .
op myObj : -> Oid .
ops in:_ out:_ : Maybe{Oid} -> Attribute .
ops inFile:_ outFile:_ : String -> Attribute .
op run : String String -> Configuration .
vars Text Original Copy : String .
vars FHIn FHOut : Oid .
var Attrs : AttributeSet .
eq run(Original, Copy)
= <>
< myObj : myClass | in: null, inFile: Original, out: null, outFile: Copy >
openFile(fileManager, myObj, Original, "r") .
rl < myObj : myClass | in: null, outFile: Copy, Attrs >
openedFile(myObj, fileManager, FHIn)
=> < myObj : myClass | in: FHIn, Attrs >
openFile(fileManager, myObj, Copy, "w") .
rl < myObj : myClass | in: FHIn, out: null, Attrs >
openedFile(myObj, fileManager, FHOut)
=> < myObj : myClass | in: FHIn, out: FHOut, Attrs >
getLine(FHIn, myObj) .
rl < myObj : myClass | in: FHIn, out: FHOut, Attrs >
gotLine(myObj, FHIn, Text)

<!-- manual page 280 -->
268 CHAPTER 9. EXTERNAL OBJECTS AND IO
=> < myObj : myClass | in: FHIn, out: FHOut, Attrs >
if Text == ""
then closeFile(FHIn, myObj)
closeFile(FHOut, myObj)
else write(FHOut, myObj, Text)
fi .
rl < myObj : myClass | in: FHIn, out: FHOut, Attrs >
wrote(myObj, FHOut)
=> < myObj : myClass | in: FHIn, out: FHOut, Attrs >
getLine(FHIn, myObj) .
rl < myObj : myClass | in: FHIn, out: FHOut, Attrs >
closedFile(myObj, FHIn)
closedFile(myObj, FHOut)
=> none .
endm
You can then execute the program to copy a file "in.txt" with the following command:
Maude> erew run("in.txt", "out.txt") .
result Portal: <>
To allow Maude to use files it must be run with the command line flags -allow-files or
-trust. See Appendix A.1.
9.3 Directory API
The DIRECTORY module (in the file.maude file) provides an API for manipulating directories.
As pointed out in the introduction of the chapter, access to external objects represents a poten-
tial threat. Directory operations are considered dangerous, and are disabled by default. The
feature may be enabled with the command line flags -allow-dir or -trust. See Appendix A.1.
mod DIRECTORY is
including CONFIGURATION .
protecting NAT .
protecting STRING .
sort EntryType EntryType? .
subsort EntryType < EntryType? .
op directory : Nat -> Oid [ctor] .
ops file directory socket pipe charDevice blockDevice : -> EntryType [ctor] .
op symbolicLink : String -> EntryType [ctor] .
op endOfDirectory : -> EntryType? [ctor] .
op openDirectory : Oid Oid String -> Msg [ctor msg format (b o)] .
op openedDirectory : Oid Oid Oid -> Msg [ctor msg format (m o)] .
op closeDirectory : Oid Oid -> Msg [ctor msg format (b o)] .
op closedDirectory : Oid Oid -> Msg [ctor msg format (m o)] .
op getDirectoryEntry : Oid Oid -> Msg [ctor msg format (b o)] .
op gotDirectoryEntry : Oid Oid String EntryType? -> Msg [ctor msg format (m o)] .
op makeDirectory : Oid Oid String -> Msg [ctor msg format (b o)] .
op madeDirectory : Oid Oid -> Msg [ctor msg format (m o)] .
op removeDirectory : Oid Oid String -> Msg [ctor msg format (b o)] .
op removedDirectory : Oid Oid -> Msg [ctor msg format (m o)] .

<!-- manual page 281 -->
9.4. SOCKETS 269
op directoryError : Oid Oid String -> Msg [ctor msg format (m o)] .
op directoryManager : -> Oid [special (...)] .
endm
Upon the reception of a openDirectory by the directoryManager external object, if suc-
cessful, results in opening the named directory. As a response, the directoryManager object
sends a openedDirectory message with the identifier of an external object representing such
directory. The identifier of the directory external object is of the form directory(N ), with N
a natural number.
Directory objects accept two kinds of messages:
• A closeDirectory message deletes the external object, frees any resources associated
with it, and produces a closedDirectory reply.
• A getDirectoryEntry message tries to read the next entry in the directory. If an entry is
successfully read, a gotDirectoryEntryreply is produced, with the name of the directory
entry (a string, third argument) and the entry type (a value of type EntryType?). The
EntryType? sort takes on the following values:
– file if a regular file,
– directory if a directory,
– socket if a Unix domain socket,
– pipe if a named pipe (aka FIFO),
– charDevice if a character device (e.g., a terminal),
– blockDevice if a block device (e.g., a hard drive), or
– symbolLink(T ) if a symbolic link pointing at T.
If there are no more entries in the directory, a gotDirectoryEntry reply is produced
where the string argument is"" and the entry type argument is endOfDirectory. Note
that endOfDirectory resides in a larger type, EntryType? than the actual entry types,
EntryType, to facilitate writing patterns that only match the latter.
A makeDirectory message sent to directoryManager, if successful, results in the creation
of an empty directory at the given path and returns a madeDirectory message.
AremoveDirectorymessagesenttodirectoryManager, ifsuccessful, resultsintheremoval
of an empty directory at the given path and returns a removedDirectory message. Nonempty
directories cannot be removed.
9.4 Sockets
The sockets external objects are accessed using the messages declared in the following SOCKET
module, included in the file socket.maude which is part of the Maude distribution.
mod SOCKET is
protecting STRING .
including CONFIGURATION .
op socket : Nat -> Oid [ctor] .
op createClientTcpSocket : Oid Oid String Nat -> Msg
[ctor msg format (b o)] .
op createServerTcpSocket : Oid Oid Nat Nat -> Msg
[ctor msg format (b o)] .
op createdSocket : Oid Oid Oid -> Msg [ctor msg format (m o)] .

<!-- manual page 282 -->
270 CHAPTER 9. EXTERNAL OBJECTS AND IO
op acceptClient : Oid Oid -> Msg [ctor msg format (b o)] .
op acceptedClient : Oid Oid String Oid -> Msg
[ctor msg format (m o)] .
op send : Oid Oid String -> Msg [ctor msg format (b o)] .
op sent : Oid Oid -> Msg [ctor msg format (m o)] .
op receive : Oid Oid -> Msg [ctor msg format (b o)] .
op received : Oid Oid String -> Msg [ctor msg format (m o)] .
op closeSocket : Oid Oid -> Msg [ctor msg format (b o)] .
op closedSocket : Oid Oid String -> Msg [ctor msg format (m o)] .
op socketError : Oid Oid String -> Msg [ctor msg format (r o)] .
op socketManager : -> Oid [special (...)] .
endm
Currently only IPv4 TCP sockets are supported; other protocol families and socket types
may be added in the future. The external object named by the constant socketManager is a
factory for socket objects.
To create a client socket, you send socketManager a message
createClientTcpSocket(socketManager, ME, ADDRESS, PORT)
where MEis the name of the object the reply should be sent to, ADDRESSis the name of the server
you want to connect to (say “www.google.com”), and PORT is the port you want to connect to
(say 80 for HTTP connections). You may also specify the name of the server as an IPv4 dotted
address or as “localhost” for the same machine where the Maude system is running on.
The reply will be either
createdSocket(ME, socketManager, NEW-SOCKET-NAME)
or
socketError(ME, socketManager, REASON)
where NEW-SOCKET-NAME is the name of the newly created socket and REASON is the operating
system’s terse explanation of what went wrong.
You can then send data to the server with a message
send(SOCKET-NAME, ME, DATA)
which elicits either
sent(ME, SOCKET-NAME)
or
closedSocket(ME, SOCKET-NAME, REASON)
Notice that all errors on a client socket are handled by closing the socket.
Similarly, you can receive data from the server with a message
receive(SOCKET-NAME, ME)
which elicits either
received(ME, SOCKET-NAME, DATA)

<!-- manual page 283 -->
9.4. SOCKETS 271
or
closedSocket(ME, SOCKET-NAME, REASON)
When you are done with the socket, you can close it with a message
closeSocket(SOCKET-NAME, ME)
with reply
closedSocket(ME, SOCKET-NAME, "")
Once a socket has been closed, its name may be reused, so sending messages to a closed
socket can cause confusion and should be avoided.
Notice that TCP does not preserve message boundaries, so sending "one" and "two" might
be received as "on"and "etwo". Delimiting message boundaries is the responsibility of the next
higher-level protocol, such as HTTP. We will present an implementation of buffered sockets in
Section 9.4.3 which solves this problem.
9.4.1 An HTTP/1.0 client example
The following modules implement an updated version of the five rule HTTP/1.0 client from
the paper “Towards Maude 2.0" [22] that is now executable. The first module defines some
auxiliary operations on strings.
fmod STRING-OPS is
protecting STRING .
var S : String .
op extractHostName : String -> String .
op extractPath : String -> String .
op extractHeader : String -> String .
op extractBody : String -> String .
eq extractHostName(S)
= if find(S, "/", 0) == notFound
then S
else substr(S, 0, find(S, "/", 0))
fi .
eq extractPath(S)
= if find(S, "/", 0) == notFound
then "/"
else substr(S, find(S, "/", 0), length(S))
fi .
eq extractHeader(S)
= substr(S, 0, find(S, "\r\n\r\n", 0) + 4) .
eq extractBody(S)
= substr(S, find(S, "\r\n\r\n", 0) + 4, length(S)) .
endfm
The second module requests one web page from an HTTP server.
mod HTTP/1.0-CLIENT is
protecting STRING-OPS .
including SOCKET .
sort State .

<!-- manual page 284 -->
272 CHAPTER 9. EXTERNAL OBJECTS AND IO
ops idle connecting sending receiving closing : -> State [ctor] .
op state:_ : State -> Attribute [ctor] .
op requester:_ : Oid -> Attribute [ctor] .
op url:_ : String -> Attribute [ctor] .
op stored:_ : String -> Attribute [ctor] .
op HttpClient : -> Cid .
op httpClient : -> Oid .
op dummy : -> Oid .
op getPage : Oid Oid String -> Msg [msg ctor] .
op gotPage : Oid Oid String String -> Msg [msg ctor] .
vars H R R’ TS : Oid .
vars U S ST : String .
First, we try to connect to the server using port 80, updating the state and the requester
attribute with the new server.
rl [getPage] :
getPage(H, R, U)
< H : HttpClient |
state: idle, requester: R’, url: S, stored: "" >
=> < H : HttpClient |
state: connecting, requester: R, url: U, stored: "" >
createClientTcpSocket(socketManager, H,
extractHostName(U), 80) .
Once we are connected to the server (we have received a createdSocket message), we send
a GET message (from the HTTP protocol) requesting the page. When the message is sent, we
wait for a response.
rl [createdSocket] :
createdSocket(H, socketManager, TS)
< H : HttpClient |
state: connecting, requester: R, url: U, stored: "" >
=> < H : HttpClient |
state: sending, requester: R, url: U, stored: "" >
send(TS, H, "GET " + extractPath(U) + " HTTP/1.0\r\nHost: " +
extractHostName(U) + "\r\n\r\n") .
rl [sent] :
sent(H, TS)
< H : HttpClient |
state: sending, requester: R, url: U, stored: "" >
=> < H : HttpClient |
state: receiving, requester: R, url: U, stored: "" >
receive(TS, H) .
While the page is not complete, we receive data and append it to the string on the stored
attribute. When the page is completed, the server closes the socket, and then we show the page
information by means of the gotPage message.
rl [received] :
received(H, TS, S)
< H : HttpClient |

<!-- manual page 285 -->
9.4. SOCKETS 273
state: receiving, requester: R, url: U, stored: ST >
=> receive(TS, H)
< H : HttpClient | state: receiving,
requester: R, url: U, stored: (ST + S) > .
rl [closedSocket] :
closedSocket(H, TS, S)
< H : HttpClient |
state: receiving, requester: R, url: U, stored: ST >
=> gotPage(R, H, extractHeader(ST), extractBody(ST)) .
We use a special operator start to represent the initial configuration. It receives the server
URL we want to connect to. Notice the occurrence of the portal <> in such initial configuration.
op start : String -> Configuration .
eq start(S)
= <>
getPage(httpClient, dummy, S)
< httpClient : HttpClient | state: idle, requester: dummy,
url: "", stored: "" > .
endm
Now we can get pages from servers, say “www.google.com”, by using the following Maude
command (note the ellipsis in the output):
Maude> erew start("www.google.com") .
result Configuration: <> gotPage(dummy, httpClient, "HTTP/1.0 ... </html>")
It is also possible to have optional bounds on the erewrite command, and then use the
continuation commands to get more results, like, for example,
Maude> erew [1, 2] start("www.google.com") .
result Configuration:
<>
< httpClient : HttpClient |
state: connecting,
requester: dummy,
url: "www.google.com",
stored: "" >
createClientTcpSocket(socketManager, httpClient, "www.google.com", 80)
Maude> cont 1 .
result Configuration:
<>
< httpClient : HttpClient |
state: connecting,
requester: dummy,
url: "www.google.com",
stored: "" >
To have communication between two Maude interpreter instances, one of them must take
the server role and offer a service on a given port; generally ports below 1024 are protected.
You cannot in general assume that a given port is available for use. To create a server socket,
you send socketManager a message
createServerTcpSocket(socketManager, ME, PORT, BACKLOG)

<!-- manual page 286 -->
274 CHAPTER 9. EXTERNAL OBJECTS AND IO
where PORT is the port number and BACKLOG is the number of queue requests for connection
that you will allow (5 seems to be a good choice). The response is either
createdSocket(ME, socketManager, SERVER-SOCKET-NAME)
or
socketError(ME, socketManager, REASON)
Here SERVER-SOCKET-NAME refers to a server socket. The only thing you can do with a server
socket (other than close it) is to accept clients, by means of the following message:
acceptClient(SERVER-SOCKET-NAME, ME)
which elicits either
acceptedClient(ME, SERVER-SOCKET-NAME, ADDRESS, NEW-SOCKET-NAME)
or
socketError(ME, socketManager, REASON)
Here ADDRESS is the originating address of the client and NEW-SOCKET-NAME is the name of the
socket you use to communicate with that client. This new socket behaves just like a client
socket for sending and receiving. Note that an error in accepting a client does not close the
server socket. You can always reuse the server socket to accept new clients until you explicitly
close it.
9.4.2 Two-Way Communication
The following modules illustrate a very naive two-way communication between two Maude
interpreter instances. The issues of port availability and message boundaries are deliberately
ignored for the sake of illustration (and thus if you are unlucky this example could fail).
The first module describes the behavior of the server.
mod FACTORIAL-SERVER is
inc SOCKET .
pr CONVERSION .
op _! : Nat -> NzNat .
eq 0 ! = 1 .
eq (s N) ! = (s N) * (N !) .
op Server : -> Cid .
op aServer : -> Oid .
vars O LISTENER CLIENT : Oid .
var A : AttributeSet .
var N : Nat .
vars IP DATA S : String .
Using the following rules, the server waits for clients. If one client is accepted, the server
waits for messages from it. When the message arrives, the server converts the received data to
a natural number, computes its factorial, converts it into a string, and finally sends this string
to the client. Once the message is sent, the server closes the socket with the client.
rl [createdSocket] :
< O : Server | A > createdSocket(O, socketManager, LISTENER)
=> < O : Server | A > acceptClient(LISTENER, O) .

<!-- manual page 287 -->
9.4. SOCKETS 275
rl [acceptedClient] :
< O : Server | A > acceptedClient(O, LISTENER, IP, CLIENT)
=> < O : Server | A > receive(CLIENT, O)
acceptClient(LISTENER, O) .
rl [received] :
< O : Server | A > received(O, CLIENT, DATA)
=> < O : Server | A >
send(CLIENT, O, string(rat(DATA, 10)!, 10)) .
rl [sent] :
< O : Server | A > sent(O, CLIENT)
=> < O : Server | A > closeSocket(CLIENT, O) .
rl [closedSocket] :
< O : Server | A > closedSocket(O, CLIENT, S)
=> < O : Server | A > .
endm
The Maude command that initializes the server is as follows, where the configuration in-
cludes the portal <>.
Maude> erew <>
< aServer : Server | none >
createServerTcpSocket(socketManager, aServer, 8811, 5) .
The second module describes the behavior of the clients.
omod FACTORIAL-CLIENT is
inc SOCKET .
class Client | .
op aClient : -> Oid .
vars O CLIENT : Oid .
Using the following rules, the client connects to the server (clients must be created after the
server), sends a message representing a number,2 and then waits for the response. When the
response arrives, there are no blocking messages and rewriting ends.
rl [createdSocket] :
< O : Client | >
createdSocket(O, socketManager, CLIENT)
=> < O : Client | >
send(CLIENT, O, "6") .
rl [sent] :
< O : Client | >
sent(O, CLIENT)
=> < O : Client | >
receive(CLIENT, O) .
endom
The initial configuration for the client will be as follows, again with portal <>.
2In this quite simple example, it is always "6".

<!-- manual page 288 -->
276 CHAPTER 9. EXTERNAL OBJECTS AND IO
Maude> erew <>
< aClient : Client | none >
createClientTcpSocket(socketManager, aClient, "localhost", 8811) .
Almost everything in the socket implementation is done in a nonblocking way; so, for ex-
ample, if you try to open a connection to some webserver and that webserver takes 5 min-
utes to respond, other rewriting and transactions happen in the meanwhile as part of the
same command erewrite. The one exception is DNS resolution, which is done as part of the
createClientTcpSocket message handling and which cannot be nonblocking without special
tricks.
9.4.3 Buffered sockets
As we said before, TCP does not preserve message boundaries; to guarantee it we may use a
filter class BufferedSocket, defined in the module BUFFERED-SOCKET, which is described here.
We interact with buffered sockets in the same way we interact with sockets, with the only
difference that all messages in the module SOCKET have been capitalized to avoid confusion.
Thus, to create a client with a buffered socket, you send socketManager a message
CreateClientTcpSocket(socketManager, ME, ADDRESS, PORT)
instead of a message
createClientTcpSocket(socketManager, ME, ADDRESS, PORT).
All the messages have exactly the same declarations, the only difference being their initial
capitalization:
op CreateClientTcpSocket : Oid Oid String Nat -> Msg
[ctor msg format (b o)] .
op CreateServerTcpSocket : Oid Oid Nat Nat -> Msg
[ctor msg format (b o)] .
op CreatedSocket : Oid Oid Oid -> Msg [ctor msg format (m o)] .
op AcceptClient : Oid Oid -> Msg [ctor msg format (b o)] .
op AcceptedClient : Oid Oid String Oid -> Msg
[ctor msg format (m o)] .
op Send : Oid Oid String -> Msg [ctor msg format (b o)] .
op Sent : Oid Oid -> Msg [ctor msg format (m o)] .
op Receive : Oid Oid -> Msg [ctor msg format (b o)] .
op Received : Oid Oid String -> Msg [ctor msg format (m o)] .
op CloseSocket : Oid Oid -> Msg [ctor msg format (b o)] .
op ClosedSocket : Oid Oid String -> Msg [ctor msg format (m o)] .
op SocketError : Oid Oid String -> Msg [ctor msg format (r o)] .
Thus, apart from this small difference, we interact with buffered sockets in exactly the same
way we do with sockets, the boundary control being completely transparent to the user.
When a buffered socket is created, in addition to the socket object through which the
information will be sent, a BufferedSocket object is also created on each side of the socket
(one in each one of the configurations between which the communication is established). All
messagessentthroughabufferedsocketaremanipulatedbeforetheyaresentthroughthesocket

<!-- manual page 289 -->
9.4. SOCKETS 277
underneath. When a message is sent through a buffered socket, a mark is placed at the end of
it; the BufferedSocket object at the other side of the socket stores all messages received on a
buffer, in such a way that when a message is requested the marks placed indicate which part
of the information received must be given as the next message.
An object of class BufferedSocket has two attributes: read, of sort String, which stores
the messages read, and bState, which indicates whether the filter is idle or active.
op BufferedSocket : -> Cid [ctor] .
op read :_ : String -> Attribute [ctor gather(&)] .
op bState :_ : BState -> Attribute [ctor gather(&)] .
sort BState .
ops idle active : -> BState [ctor] .
The identifiers of the BufferedSocket objects are marked with a b operator, i.e., the buffers
associatedwithasocketSOCKEThaveidentifierb(SOCKET).NotethatthereisaBufferedSocket
object on each side of the socket, that is, there are two objects with the same identifier, but in
different configurations.
op b : Oid -> Oid [ctor] .
A buffered socket object understands capitalized versions of the messages a socket object
understands. For most of them, it just converts them into the corresponding uncapitalized
message. There are messages AcceptClient, CloseSocket, CreateServerTcpSocket, and
CreateClientTcpSocket with the same arities as the corresponding socket messages, with
the following rules.
vars SOCKET NEW-SOCKET SOCKET-MANAGER O : Oid .
vars ADDRESS IP IP’ DATA S S’ REASON : String .
var Atts : AttributeSet .
vars PORT BACKLOG N : Nat .
rl [createServerTcpSocket] :
CreateServerTcpSocket(SOCKET-MANAGER, O, PORT, BACKLOG)
=> createServerTcpSocket(SOCKET-MANAGER, O, PORT, BACKLOG) .
rl [acceptClient] :
AcceptClient(SOCKET, O)
=> acceptClient(SOCKET, O) .
rl [closeSocket] :
CloseSocket(b(SOCKET), SOCKET-MANAGER)
=> closeSocket(SOCKET, SOCKET-MANAGER) .
rl [createClientTcpSocket] :
CreateClientTcpSocket(SOCKET-MANAGER, O, ADDRESS, PORT)
=> createClientTcpSocket(SOCKET-MANAGER, O, ADDRESS, PORT) .
Note that in these cases the buffered-socket versions of the messages are just translated into
the corresponding socket messages.
A BufferedSocket object can also convert an uncapitalized message into the capitalized
one. The rule socketError shows this:
rl [socketError] :
socketError(O, SOCKET-MANAGER, REASON)
=> SocketError(O, SOCKET-MANAGER, REASON) .

<!-- manual page 290 -->
278 CHAPTER 9. EXTERNAL OBJECTS AND IO
BufferedSocket objects are created and destroyed when the corresponding sockets are.
Thus, we have rules
rl [acceptedclient] :
acceptedClient(O, SOCKET, IP’, NEW-SOCKET)
=> AcceptedClient(O, b(SOCKET), IP’, b(NEW-SOCKET))
< b(NEW-SOCKET) : BufferedSocket |
bState : idle, read : "" > .
rl [createdSocket] :
createdSocket(O, SOCKET-MANAGER, SOCKET)
=> < b(SOCKET) : BufferedSocket | bState : idle, read : "" >
CreatedSocket(O, SOCKET-MANAGER, b(SOCKET)) .
rl [closedSocket] :
< b(SOCKET) : BufferedSocket | Atts >
closedSocket(SOCKET, SOCKET-MANAGER, DATA)
=> ClosedSocket(b(SOCKET), SOCKET-MANAGER, DATA) .
Once a connection has been established, and a BufferedSocket object has been created on
each side, messages can be sent and received. When a Send message is received, the buffered
socketsendsasendmessagewiththesamedataplusamark3 toindicatetheendofthemessage.
rl [send] :
< b(SOCKET) : BufferedSocket | bState : active, Atts >
Send(b(SOCKET), O, DATA)
=> < b(SOCKET) : BufferedSocket | bState : active, Atts >
send(SOCKET, O, DATA + "#") .
rl [sent] :
< b(SOCKET) : BufferedSocket | bState : active, Atts >
sent(O, SOCKET)
=> < b(SOCKET) : BufferedSocket | bState : active, Atts >
Sent(O, b(SOCKET)) .
The key is then in the reception of messages. A BufferedSocket object is always listening
to the socket. It sends a receive message at start up and puts all the received messages in its
buffer. Notice that a buffered socket goes from idle to active in the buffer-start-up rule.
A Receive message is then handled if there is a complete message in the buffer, that is, if there
is a mark on it, and results in the reception of the first message in the buffer, which is removed
from it.
rl [buffer-start-up] :
< b(SOCKET) : BufferedSocket | bState : idle, Atts >
=> < b(SOCKET) : BufferedSocket | bState : active, Atts >
receive(SOCKET, b(SOCKET)) .
rl [received] :
< b(SOCKET) : BufferedSocket |
bState : active, read : S, Atts >
received(b(SOCKET), O, DATA)
=> < b(SOCKET) : BufferedSocket |
bState : active, read : (S + DATA), Atts >
3We use the character ‘#’ as mark; therefore, the user data sent through the sockets should not contain such
a character.

<!-- manual page 291 -->
9.5. PROCESSES 279
receive(SOCKET, b(SOCKET)) .
crl [Received] :
< b(SOCKET) : BufferedSocket |
bState : active, read : S, Atts >
Receive(b(SOCKET), O)
=> < b(SOCKET) : BufferedSocket |
bState : active, read : S’, Atts >
Received(O, b(SOCKET), DATA)
if N := find(S, "#", 0)
/\ DATA := substr(S, 0, N)
/\ S’ := substr(S, N + 1, length(S)) .
The BUFFERED-SOCKET module is used in the specification of Mobile Maude, a mobile agent
language based on Maude, which is discussed in detail in [24, Chapter 16].
9.5 Processes
Unix processes can be handled as Maude external objects through the API contained in the
process.maudefile. Aspointedoutintheintroductionofthechapter, accesstoexternalobjects
represents a potential threat. Process execution is disabled by default. The feature may be
enabled with the command line flags -allow-processes or -trust. See Appendix A.1.
The external object processManager allows us to create new processes (with the message
createProcess), with which we may directly interact.
fmod STRING-LIST is
protecting LIST{String} *
(sort NeList{String} to NeStringList, sort List{String} to StringList) .
endfm
mod PROCESS is
including SOCKET .
protecting STRING-LIST .
sorts ProcessOption ProcessOptionSet .
subsort ProcessOption < ProcessOptionSet .
op none : -> ProcessOptionSet [ctor] .
sort ExitStatus .
op normalExit : Nat -> ExitStatus [ctor] .
op terminatedBySignal : String -> ExitStatus [ctor] .
op process : Nat -> Oid [ctor] .
op createProcess : Oid Oid String StringList ProcessOptionSet -> Msg
[ctor msg format (b o)] .
op createdProcess : Oid Oid Oid Oid Oid -> Msg [ctor msg format (m o)] .
op signalProcess : Oid Oid String -> Msg [ctor msg format (b o)] .
op signaledProcess : Oid Oid -> Msg [ctor msg format (m o)] .
op waitForExit : Oid Oid -> Msg [ctor msg format (b o)] .
op exited : Oid Oid ExitStatus -> Msg [ctor msg format (m o)] .

<!-- manual page 292 -->
280 CHAPTER 9. EXTERNAL OBJECTS AND IO
op processError : Oid Oid String -> Msg [ctor msg format (r o)] .
op processManager : -> Oid [special (...)] .
endm
New processes are created with the createProcess message, which has five arguments:
• As usual, the first argument is the addressee of the message, in this case the special builtin
object processManager, a constant of sort Oid;
• the second argument is the Oid of the requesting object, to which the response will be
sent;
• the third argument is a String with the system command that executes the new process;
• the fourth argument is a StringList with the arguments to the command; and
• the fifth argument is a ProcessOptionSet, which is reserved for future use (currently we
will use none).
On success, the reply to a createProcess message is a message createdProcess, which
also carries five arguments, all of type Oid:
• the requesting object,
• the sender object (the processManager object),
• the identifier of the new process object, which uses the process constructor,
• the identifier of the new socket object for stdin/stdout, and
• the identifier of the new socket object for stderr.
One can communicate with the new process using its stdin/stdout socket and the normal
socket interface defined in socket.maude (see Section 9.4). Error messages can be read on the
stderr socket. It is probably good practice to always have receives pending on both sockets so
the process is not stalled due to socket buffers filling. The stderr socket ignores send messages.
Maude will stop rewriting and return to the command line as soon as there are no rewrites
possible and no external events pending that could change the rewrite state, even if the newly
created process is still running. It is therefore good practice to wait for a process to exit,
if only to avoid filling the process table with zombies. This may be done by sending the
process a waitForExit message. If such a message is sent, Maude will not return to the
command line until the process actually exits, which is what happens when there are no further
pending external events. When the process finally exits, the reply message is exited, with
the last argument being either normalExit with an 8-bit exit code for a normal exit or a
terminatedBySignal with the name of the signal in the case of termination by signal. The
waitForExit message is nonblocking, so it can be sent any time after the createdProcess
message is received. A second waitForExit message to the same process will not be accepted.
When a process has been waited for, its stdin/stdout and stderr are closed. If processes are
not waited to be closed, it is the user’s responsibility to close the sockets to avoid leaking file
descriptors. Onewaytodothisistokeepreceivinganyfinalcharactersfromtheprocessuntilan
end-of-file or error condition arises and Maude returns a closedSocket message. Alternatively,
sockets can be closed by explicitly sending them closeSocket messages.
Different executables have different conventions for telling them to quit. You can send a
process the empty string to cause an EOF condition on its stdin (see below). You can also send
a signal to a process using the signalProcess message, whose third argument is a string with
the name of the Unix signal.4 The process responds with a signaledProcess message.
ThemessageprocessErrorisusedasthereplyforerrorscomingfromtheoperatingsystem.
4Not every signal is supported by every platform and not every signal makes sense to send to a child process.
The following POSIX signals are supported on all Unix platforms: SIGHUP, SIGINT, SIGQUIT, SIGKILL, SIGALRM,
SIGTERM, SIGSTOP, and SIGCONT; everything else should be considered platform dependent.

<!-- manual page 293 -->
9.5. PROCESSES 281
In particular a processErrorreply is generated if the child process cannot execute the specified
executable.
We provide simple examples illustrating these features in the following sections.
9.5.1 A desk calculator process
Our first example invokes the dc (desk calculator) program to do some calculations and then
kills it with a SIGTERM signal. To interact with the external objects we use an object of class
User with attributes state (to keep track of the state the object is in), process (to keep the
Oid of the external object), io and err (to keep the socket objects’ identifiers), and result (to
store the response). Please, see inlined comments explaining the rules.
omod PROCESS-DC is
inc PROCESS .
inc MAYBE{Oid} * (op maybe{Oid} to null) .
class User | state : Nat, process : Maybe{Oid}, io : Maybe{Oid},
err : Maybe{Oid}, result : String .
op me : -> Oid .
vars X P ERR IO : Oid .
vars Result Str : String .
rl < X : User | state : 1, process : null, io : null, err : null >
createdProcess(X, processManager, P, IO, ERR)
=> < X : User | state : 2, process : P, io : IO, err : ERR >
send(IO, X, "10 16 + p\n")
waitForExit(P, X) .
rl < X : User | state : 2, io : IO >
sent(X, IO)
=> < X : User | state : 3, io : IO >
receive(IO, X) .
rl < X : User | state : 3, io : IO, result : "" >
received(X, IO, Result) ---- Result --> "26\n"
=> < X : User |
state : 4, io : IO, result : substr(Result, 0, sd(length(Result), 1)) >
send(IO, X, "4 * p\n") .
rl < X : User | state : 4, io : IO >
sent(X, IO)
=> < X : User | state : 5, io : IO >
receive(IO, X) .
rl < X : User | state : 5, process : P, io : IO, result : Str >
received(X, IO, Result) ---- Result --> "104\n"
=> < X : User | state : 6, process : P,
io : IO, result : substr(Result, 0, sd(length(Result), 1)) >
signalProcess(P, X, "SIGTERM") .
rl < X : User | state : 6, process : P >
signaledProcess(X, P)
exited(X, P, normalExit(0))
=> < X : User | state : 7, process : P > .
endom
We can run the above program with the following erewrite command:
Maude> erew <>

<!-- manual page 294 -->
282 CHAPTER 9. EXTERNAL OBJECTS AND IO
< me : User | state : 1,
process : null,
io : null,
err : null,
result : "" >
createProcess(processManager, me, "dc", nil, none) .
rewrites: 12 in 2ms cpu (9ms real) (4175 rewrites/second)
result Configuration:
<>
< me : User | state : 7,
process : process(33376),
io : socket(3),
err : socket(5),
result : "104" >
To allow Maude to use processes it must be run with the command line flags -allow-processes
or -trust. See Appendix A.1. The execution terminates with the User object in its final state,
with "104" as final result of the operations carried out by the dc process. Notice that when
the current erewrite context is destroyed (say, by running a new rewrite command), Maude
will kill any processes and close any sockets that correspond to Maude external objects in that
context.
9.5.2 Python and Maude processes
Each application has its own form of interaction. In this section we present a Maude program
that interacts with a second Maude process, and, to analyze its output, it uses a Python
program that is run on the Python 3 interpreter. Specifically, the program runs an external
Maude process in which it loads the process-dc.maude file presented in the previous section.
Asshownthere, theprogramusesthedctodoacalculation. Sincetheoutputisrathercomplex,
we use a Python program to process the given output and extract the part we are interested
in. The output is processed in a Python interpreter process using the function select_text.
The string "104" is returned as result.
Although just for illustration purposes in this case, the use of other languages applications
may be convenient in some of our programs. In this case, the use of regular expressions for the
manipulation of strings comes quite handy. Our Python function select_text is very simple;
it uses the functionality provided in the re module for searching a string s, passed as first
argument, for a regular expression with the exact pattern we are interested in, which is given
as second argument p:
def select_text(s, p):
result = re.compile(p).search(s)
if result:
return result.group(1)
else:
raise Exception("not␣found")
As for the example in Section 9.5.1, we use an object to keep track of the process and
information of the different sockets. Since we are going to have two of these processes, we can
introduce the appropriate definitions as a generic class.
omod PROCESS-PROXY is
inc PROCESS .
inc MAYBE{Oid} * ( op maybe{Oid} to null ) .

<!-- manual page 295 -->
9.5. PROCESSES 283
class Proxy | state : Nat, process : Maybe{Oid}, io : Maybe{Oid}, err : Maybe{Oid} .
endom
In this program, we assume that both the Maude system (maude.darwin64) and the Python
interpreter (python3) are in your path, and that the process-dc.maude file from the previous
section is in your working directory. If this is not the case, please change the code below
appropriately. A PATH constant is given for the location of Maude’s executable, which should
be changed depending on your setting.
The functionality provided by the select_text Python function may be useful in different
contexts. To make it available as an auxiliary function, we show below the PROCESS-PYTHON
module, which provides a function string-extract that extracts a substring from a given
string that matches a given pattern. Specifically, given a string-extract(X, Y, Str, Pat)
message, it returns either string-extracted(Y, X, Result), with the matched substring, or
string-extract-error(Y, X, Result), if any error happens.
omod PROCESS-PYTHON is
inc PROCESS-PROXY .
class Python | string : String, pattern : String, requester : Maybe{Oid} .
subclass Python < Proxy .
op string-extract : Oid Oid String String -> Msg .
ops string-extracted string-extract-error : Oid Oid String -> Msg .
vars X Y P ERR IO : Oid .
vars Result Str Pat : String .
rl string-extract(X, Y, Str, Pat)
=> < X : Python | state : 1, process : null, io : null, err : null,
requester : Y, string : Str, pattern : Pat >
createProcess(processManager, X, "python3", "-iu" "string_extract.py", none) .
rl < X : Python | state : 1, process : null, io : null, err : null,
string : Str, pattern : Pat >
createdProcess(X, processManager, P, IO, ERR)
=> < X : Python | state : 2, process : P, io : IO, err : ERR >
send(IO, X, "select_text(’’’" + Str + "’’’, ’" + Pat + "’)\n")
waitForExit(P, X) .
rl < X : Python | state : 2, io : IO, err : ERR >
sent(X, IO)
=> < X : Python | state : 3, io : IO, err : ERR >
receive(IO, X)
receive(ERR, X) .
rl < X : Python | state : 3, io : IO, requester : Y > ---- output received
received(X, IO, Result) ---- Result --> "’104’\n"
=> < X : Python | state : 4, io : IO, requester : Y >
string-extracted(Y, X, substr(Result, 1, sd(length(Result), 3)))
send(IO, X, "quit()\n") .
rl < X : Python | state : 4, io : IO > ---- quit sent
sent(X, IO)
=> < X : Python | state : 5, io : IO > .
rl < X : Python | err : ERR, requester : Y > ---- error
received(X, ERR, Result)
=> < X : Python | err : ERR, requester : Y > ---- error msg might be empty
if Result == ">>> "

<!-- manual page 296 -->
284 CHAPTER 9. EXTERNAL OBJECTS AND IO
then none
else string-extract-error(Y, X, Result)
fi .
rl < X : Python | state : 5, process : P > exited(X, P, normalExit(0))
=> none .
---- process terminated
endom
The rules are explained in inline comments, but several issues are worth pointing out:
• Although string-extract is declared as a message, notice that its first argument is the
identifier used to create the proxy.
• The string and the pattern need to be kept by the wrapper until the reception of the
createdProcess message.
• The Python process is started in interactive mode (using the -i flag), and the file
string_extract.py with the above select_text function is also passed as a command
argument.
• In Python, strings may use either single quotes (’), quotes ("), or triple quotes ("""
or ’’’). Since the output from the Maude process may contain quotes and new line
characters, the string to search in is passed using triple single quotes. The pattern is
passed using single quotes, assuming that it does not contain new line characters. Quotes
could also be used in both cases, but then the string on which the search is performed
should have to be manipulated before it is sent as an argument of the Python command.
Remember that quotes inside strings must be double quoted, i.e., a string like "\"hi\""
must be sent as argument of the function as "select_text(\"\\\"hi\\\"\")". Notice
that, since to put a quote inside a string you use \", and to put a backslash inside a string
you use \\, then, to put \" inside a string you must use \\\".
• The process is terminated using its quit command. Once the process is terminated, the
wrapper is removed.
The following module uses the above string-extract function to extract the result from
the execution of the process-dc.maude program.
Although the rules are explained in inlined comments below, there are a couple of issues
worth pointing out:
• The process-dc.maude file is passed as an argument of the maude.darwin64 command
when the createProcess is sent. Notice also that Maude has to be invoked with the
-trust flag to run external processes.
• Maude gives a welcome header when started, and when a command is sent to it, it first
acknowledges the command being executed, and then gives the actual output when it
finishes.
• In this case, we use a constant init to initiate the execution. The output and potential
error messages are given with respective operators result and error.
omod PROCESS-MAUDE is
---- Maude’s path, empty string if in the system’s path or in the same directory
op PATH : -> String .
eq PATH = "/Users/duran/Maude/maude-3.4" .
inc PROCESS-PYTHON .
msgs result error : String -> Msg .
op init : -> Configuration .
ops me python : -> Oid .

<!-- manual page 297 -->
9.5. PROCESSES 285
vars X Y P ERR IO : Oid .
vars Result Str Pat : String .
---- the program begins by creating the Maude process
rl init
=> < me : Proxy | state : 1, process : null, io : null, err : null >
createProcess(processManager, me,
PATH + "/maude.darwin64", "-trust" "process-dc.maude", none) .
---- Upon creation of the Maude process, it requests its banner
rl < X : Proxy | state : 1, process : null, io : null, err : null >
createdProcess(X, processManager, P, IO, ERR)
=> < X : Proxy | state : 2, process : P, io : IO, err : ERR >
receive(IO, X)
waitForExit(P, X) .
---- When the banner is received, the command is sent
---- This is the same command used in the previous section
rl < X : Proxy | state : 2, io : IO >
received(X, IO, Result) ---- Maude’s header
=> < X : Proxy | state : 3, io : IO >
send(IO, X,
"erew <> " +
"< me : User | state : 1, process : null, " +
" io : null, err : null, result : \"\" > " +
"createProcess(processManager, me, \"dc\", nil, none) .\n") .
---- When the sending is confirmed, a request for input is sent
rl < X : Proxy | state : 3, io : IO >
sent(X, IO)
=> < X : Proxy | state : 4, io : IO >
receive(IO, X) .
---- Maude first sends the command being executed
---- In this case we send a receive message on both channels
rl < X : Proxy | state : 4, io : IO, err : ERR >
received(X, IO, Result) ---- command being executed
=> < X : Proxy | state : 5, io : IO, err : ERR >
receive(IO, X) ---- request output
receive(ERR, X) . ---- and potential error message
---- Once an answer is received, a quit command is sent to the Maude
---- process to terminate it, and a string-extract message is delivered
rl < X : Proxy | state : 5, io : IO > ---- result from ok execution
received(X, IO, Result)
=> < X : Proxy | state : 6, io : IO >
send(IO, X, "q\n")
string-extract(python, X, Result, "result : \"([^\"]*)\"") .
---- the response from the string-extract message is handled
rl < X : Proxy | state : 6, io : IO > ---- result from ok execution
string-extracted(X, Y, Result)
=> < X : Proxy | state : 7, io : IO > ---- result from ok execution
result(Result) .
rl < X : Proxy | > ---- error execution
string-extract-error(X, Y, Result)
=> < X : Proxy | >
error(Result) .
---- non-empty error messages are forwarded

<!-- manual page 298 -->
286 CHAPTER 9. EXTERNAL OBJECTS AND IO
rl < X : Proxy | err : ERR > ---- error
received(X, ERR, Result)
=> < X : Proxy | >
if Result == "" then none else error(Result) fi . ---- no error message
---- sent and exited messages are consumed
rl < X : Proxy | state : 7, io : IO, process : P >
sent(X, IO)
exited(X, P, normalExit(0))
=> none .
endom
The execution of the program is then as follows.
Maude> erew in PROCESS-MAUDE : <>
init .
rewrites: 32 in 5ms cpu (61ms real) (5457 rewrites/second)
result Configuration: <> result("104")
9.6 Time API
The TIME module provides an API for date and time as well as for timers. Three examples are
given in subsections.
mod TIME is
protecting INT .
protecting STRING .
including CONFIGURATION .
sorts TimerMode Date Time TimeZoneInfo .
ops oneShot periodic : -> TimerMode [ctor] .
op _\_\_ : NzNat NzNat NzNat -> Date [ctor] .
op _:_:_ : Nat Nat Nat -> Time [ctor] .
op [_,_,_,_] : String String Int Int -> TimeZoneInfo [ctor] .
op timer : Nat -> Oid [ctor] .
op getTimeSinceEpoch : Oid Oid -> Msg [ctor msg format (b o)] .
op gotTimeSinceEpoch : Oid Oid Nat -> Msg [ctor msg format (m o)] .
op getDateAndTime : Oid Oid Nat -> Msg [ctor msg format (b o)] .
op gotDateAndTime : Oid Oid Date Time NzNat NzNat -> Msg [ctor msg format (m o)] .
op getLocalDateAndTime : Oid Oid Nat -> Msg [ctor msg format (b o)] .
op gotLocalDateAndTime : Oid Oid Date Time NzNat NzNat TimeZoneInfo -> Msg
[ctor msg format (m o)] .
op createTimer : Oid Oid -> Msg [ctor msg format (b o)] .
op createdTimer : Oid Oid Oid -> Msg [ctor msg format (m o)] .
op startTimer : Oid Oid TimerMode NzNat -> Msg [ctor msg format (b o)] .
op startedTimer : Oid Oid -> Msg [ctor msg format (m o)] .
op timeOut : Oid Oid -> Msg [ctor msg format (r o)] .
op stopTimer : Oid Oid -> Msg [ctor msg format (b o)] .
op stoppedTimer : Oid Oid -> Msg [ctor msg format (m o)] .

<!-- manual page 299 -->
9.6. TIME API 287
op deleteTimer : Oid Oid -> Msg [ctor msg format (b o)] .
op deletedTimer : Oid Oid -> Msg [ctor msg format (m o)] .
op timeError : Oid Oid String -> Msg [ctor msg format (r o)] .
op timeManager : -> Oid [special (...)] .
***
*** Time unit conversion.
***
var N : Nat .
op _us : Nat -> Nat .
eq N us = 1000 * N .
op _ms : Nat -> Nat .
eq N ms = 1000000 * N .
op _seconds : Nat -> Nat .
eq N seconds = 1000000000 * N .
endm
There is an external object timeManager. Sending to it the message getTimeSinceEpoch
results in a reply gotTimeSinceEpoch where the last argument is the number of nanoseconds
since the Unix Epoch (00:00:00 UTC on 1 January 1970). Computers synchronizing their time
via the Network Time Protocol (NTP) will only be accurate to a few tens of milliseconds.
Conversion from nanoseconds since the Unix Epoch to more conventional date and time
representations is handled by the message pairs:
op getDateAndTime : Oid Oid Nat -> Msg [ctor msg format (b o)] .
op gotDateAndTime : Oid Oid Date Time NzNat NzNat -> Msg [ctor msg format (m o)] .
and
op getLocalDateAndTime : Oid Oid Nat -> Msg [ctor msg format (b o)] .
op gotLocalDateAndTime : Oid Oid Date Time NzNat NzNat TimeZoneInfo -> Msg
[ctor msg format (m o)] .
The former converts to Coordinated Universal Time (UTC) while the latter converts to the
local time zone.
Dates are represented using the operator
op _\_\_ : NzNat NzNat NzNat -> Date [ctor] .
where the arguments are year (1969,...), month (1 - 12), and day (1 - 31).
Times are represented using the operator
op _:_:_ : Nat Nat Nat -> Time [ctor] .
where the arguments are hour (0 - 23), minute (0 - 59), and second (0 - 60). The value of 60
for the second only arises with leap seconds.
The next two arguments in both gotDateAndTime and gotLocalDateAndTime are nonzero
natural numbers representing day-of-the-year (1 - 366), with 366 only arising in leap years, and
day-of-the-week (1 = Sunday, ..., 7 = Saturday).
The last argument of the gotLocalDateAndTime message is a value of type TimeZoneInfo,
which is represented by operator
op [_,_,_,_] : String String Int Int -> TimeZoneInfo [ctor] .

<!-- manual page 300 -->
288 CHAPTER 9. EXTERNAL OBJECTS AND IO
The first two arguments are the abbreviations for the names of the standard time and daylight
saving time for the time zone. The second of these can be the empty string if there is no
daylight saving time for the time zone. The next argument is the time zone specified as an
offset in seconds from UTC. This number is negative for time zones west of the Prime Meridian
to the International Date Line and positive for time zones east of the Prime Meridian to the
International Date Line. Note that this number of seconds does not take daylight saving into
account. The final Int argument indicates whether daylight savings time is in effect. A positive
value indicates that the daylight savings time in effect, 0 indicates that it is not in effect, and
a negative value indicates that the information is not available.
Note that both getDateAndTime and getLocalDateAndTime take as third argument a non-
negative number of nanoseconds since the Unix Epoch. The upper bound on the number of
nanoseconds that can be converted depends on the underlying platform.
As for other external objects, all messages addressed to valid object will be consumed,
returning a timeError message in case of error.
op timeError : Oid Oid String -> Msg [ctor msg format (r o)] .
Possible error messages include "Time out-of-range." and "Bad time.". It is also possible
to get an error message from the underlying C library if the year cannot be represented.
9.6.1 Timers
As well as timeManager, there are more ephemeral external objects called timers that can
be used to generate timeOut messages at a given future time. Timer objects are created by
communicating with timeManager using the message pair:
op createTimer : Oid Oid -> Msg [ctor msg format (b o)] .
op createdTimer : Oid Oid Oid -> Msg [ctor msg format (m o)] .
The final argument of the createdTimer message is the identifier of a timer object. These
identifiers are represented using a timeroperator, and therefore they are of the form timer(n ).
A timer is started using the message pair:
op startTimer : Oid Oid TimerMode NzNat -> Msg [ctor msg format (b o)] .
op startedTimer : Oid Oid -> Msg [ctor msg format (m o)] .
The TimerMode sort is defined with two possible values, namely oneShot and periodic. If
the startTimer message is sent with oneShot as third argument, the timer will send a single
timeOut message. The timer then enters a quiescent state, from which it may be restarted at
a later time by another startTimer message. If startTimer message is sent with periodic
as third argument, the timer sends a series of timeOut messages until it is stopped, deleted or
restarted. The last argument of the startTimer message (NzNat) specifies the time to count
down from, and which, in periodic mode is reset after it reaches zero.
In periodic mode, if a given timeOut message is delayed for some reason, it does not impact
the time at which future timeOut messages will be delivered. Note that, in periodic mode, if
the countdown start time is too short the configuration can quickly fill with timeOut messages.
It can even be the case that if the period is too short, it will be hard to enter two control-C
interrupts on the same suspension to cause Maude to stop.
If a startTimer message is sent to a running timer, its current countdown time and mode
are forgotten and it restarts fresh with the new parameters.
A running timer can be returned to quiescent state with the message pair:
op stopTimer : Oid Oid -> Msg [ctor msg format (b o)] .
op stoppedTimer : Oid Oid -> Msg [ctor msg format (m o)] .

<!-- manual page 301 -->
9.6. TIME API 289
Finally a timer (running or quiescent) can be deleted with the message pair:
op deleteTimer : Oid Oid -> Msg [ctor msg format (b o)] .
op deletedTimer : Oid Oid -> Msg [ctor msg format (m o)] .
Note that there is necessarily a race condition when sending a message to a running timer, since
the timeOut message may be emitted before the incoming message is received. For this reason
it is legal to send a stopTimer message to a quiescent timer since it may have been running in
oneShot mode and may have just timed out.
The shortest allowed start time is 1 nanosecond. The longest allowed start time is 2ˆ63 - 1
nanoseconds or about 292 years.
As with other external objects, Maude only returns to the command line after all pending
external messages have arrived and there are no local rewrites available.
The strings of timeError include "Bad timer start value." and "Bad timer mode.".
9.6.2 Current UTC and local time
With the following module TIME-UTC-LOCAL we get the number of nanoseconds since the Unix
Epoch and converts it into both UTC and local time.
load time
mod TIME-UTC-LOCAL is
inc TIME .
op Class : -> Cid .
ops me : -> Oid .
op run : -> Configuration .
eq run
= <>
< me : Class | none >
getTimeSinceEpoch(timeManager, me) .
vars O O2 : Oid .
var N : Nat .
rl < O : Class | none >
gotTimeSinceEpoch(O, O2, N)
=> < O : Class | none >
getDateAndTime(O2, O, N)
getLocalDateAndTime(O2, O, N).
endm
Maude> erew run .
erewrite in TEST : run .
rewrites: 2 in 0ms cpu (0ms real) (7092 rewrites/second)
result Configuration:
<>
< me : Class | none >
gotDateAndTime(me, timeManager, 2022 \ 3 \ 18, 17 : 35 : 52, 77, 6)
gotLocalDateAndTime(me, timeManager,
2022 \ 3 \ 18, 18 : 35 : 52, 77, 6, ["CET","CEST",3600,0])
As we can see in the output of the erew command, when the command was executed, the
date was 2022 \ 3 \ 18 (March 18th, 2022), which corresponds to the 77th day of the year,
a Friday, and the time 17 : 35 : 52. The command was run on a machine in a place with
Central European Standard Time (CET). CEST stands for Central European Summer Time,

<!-- manual page 302 -->
290 CHAPTER 9. EXTERNAL OBJECTS AND IO
which begins in the last Sunday of March. The 0 indicates that the daylight saving time is
currently not active. The offset with respect to UTC is 3600 seconds (one hour).
9.6.3 Measuring an interval of time
The following example measures the time between fair rewriting traversals. It just sends two
getTimeSinceEpoch messages in succession and calculates the time between them. The time
of the first response is stored in a prevTime attribute to be able to calculate the difference once
the second response arrives.
load time
omod TIME-INTERVAL is
inc TIME .
class Class | prevTime : Nat, interval : Nat .
op me : -> Oid .
op run : -> Configuration .
eq run
= <>
< me : Class | prevTime : 0, interval : 0 >
getTimeSinceEpoch(timeManager, me) .
vars O O2 : Oid . var N N2 : NzNat .
rl < O : Class | prevTime : 0 >
gotTimeSinceEpoch(O, O2, N)
=> < O : Class | prevTime : N >
getTimeSinceEpoch(timeManager, me) .
rl < O : Class | prevTime : N >
gotTimeSinceEpoch(O, O2, N2)
=> < O : Class | interval : (N2 - N) > .
endom
Maude> erew run .
erewrite in TIME-INTERVAL : run .
rewrites: 4 in 0ms cpu (0ms real) (44444 rewrites/second)
result Configuration:
<>
< me : Class | prevTime : 1712172117320830000,
interval : 39000 >
The result shows that the time between rule applications was 39000 nanoseconds.
9.6.4 An example with timers
In the following example, three timers are created. The first is periodic, every 500 ms. The
other two are one-shot timers, with timeouts 1250 ms and 3250 ms, respectively. When each
timeOut message is received, a message is printed on the standard output.
load time
load file
fmod MAYBE{X :: TRIV} is
sort Maybe{X} .
subsort X$Elt < Maybe{X} .
op maybe{X} : -> Maybe{X} [ctor] .

<!-- manual page 303 -->
9.6. TIME API 291
endfm
view Oid from TRIV to CONFIGURATION is
sort Elt to Oid .
endv
omod TEST is
inc TIME .
inc STD-STREAM .
inc MAYBE{Oid} * (op maybe{Oid} to null) .
class Class | firstTimer : Maybe{Oid}, secondTimer : Maybe{Oid}, thirdTimer : Maybe{Oid} .
op me : -> Oid .
op run : -> Configuration .
eq run
= <>
< me : Class | firstTimer : null, secondTimer : null, thirdTimer : null >
createTimer(timeManager, me) .
vars O O2 O3 O4 O5 : Oid .
var N : Nat .
rl < O : Class | firstTimer : null >
createdTimer(O, O2, O3)
=> < O : Class | firstTimer : O3 >
createTimer(timeManager, me) .
rl < O : Class | firstTimer : O3, secondTimer : null >
createdTimer(O, O2, O4)
=> < O : Class | secondTimer : O4 >
createTimer(timeManager, me) .
rl < O : Class | firstTimer : O3, secondTimer : O4, thirdTimer : null >
createdTimer(O, O2, O5)
=> < O : Class | thirdTimer : O5 >
startTimer(O3, me, periodic, 500 ms)
startTimer(O4, me, oneShot, 1250 ms)
startTimer(O5, me, oneShot, 3250 ms).
rl < O : Class | firstTimer : O3 >
timeOut(O, O3)
=> < O : Class | >
write(stdout, O, "time out for first timer\n") .
rl < O : Class | secondTimer : O4 >
timeOut(O, O4)
=> < O : Class | >
write(stdout, O, "time out for second timer\n") .
rl < O : Class | firstTimer : O3, thirdTimer : O5 >
timeOut(O, O5)
=> < O : Class | >
write(stdout, O, "time out for third timer\n")
stopTimer(O3, O) .
rl < O : Class | firstTimer : O3, secondTimer : O4, thirdTimer : O5 >
stoppedTimer(O, O3)
=> < O : Class | none >
deleteTimer(O3, O) deleteTimer(O4, O) deleteTimer(O5, O) .
endom
Maude> erew run .

<!-- manual page 304 -->
292 CHAPTER 9. EXTERNAL OBJECTS AND IO
erewrite in TEST : run .
time out for first timer
time out for first timer
time out for second timer
time out for first timer
time out for first timer
time out for first timer
time out for first timer
time out for third timer
rewrites: 19 in 1ms cpu (3255ms real) (12675 rewrites/second)
result Configuration:
<>
startedTimer(me, timer(0))
startedTimer(me, timer(1))
startedTimer(me, timer(2))
deletedTimer(me, timer(0))
deletedTimer(me, timer(1))
deletedTimer(me, timer(2))
wrote(me, stdout)
wrote(me, stdout)
wrote(me, stdout)
wrote(me, stdout)
wrote(me, stdout)
wrote(me, stdout)
wrote(me, stdout)
wrote(me, stdout)
< me : Class | firstTimer : timer(0), secondTimer : timer(1), thirdTimer : timer(2) >
The output shows the messages printed in the terminal and the final configuration. We can
observe:
• The first timer gets two timeouts (at times 500 and 1000) before the second one gets its
(at time 1500).
• The first timer gets four more timeouts (at times 1500, 2000, 2500, and 3000) before the
third one gets its (at time 3250).
• The final configuration contains several messages indicating that the timers were started
and deleted, and that several messages were written on standard output.
• The execution time was 3255 ms, 5 ms more than the required by the third timer, which
triggers the deletion of the timers.
9.7 Pseudo-random number generator objects
The PRNG module provides an API for pseudo-random number generator (PRNG) objects.
mod PRNG is
protecting STRING .
including CONFIGURATION .
op prng : Nat -> Oid [ctor] .
op createPrng : Oid Oid String -> Msg [ctor msg format (b o)] .
op createdPrng : Oid Oid Oid -> Msg [ctor msg format (m o)] .
op setSeed : Oid Oid Nat -> Msg [ctor msg format (b o)] .

<!-- manual page 305 -->
9.7. PSEUDO-RANDOM NUMBER GENERATOR OBJECTS 293
op seedSet : Oid Oid -> Msg [ctor msg format (m o)] .
op getNext : Oid Oid -> Msg [ctor msg format (b o)] .
op gotNext : Oid Oid Nat -> Msg [ctor msg format (m o)] .
op deletePrng : Oid Oid -> Msg [ctor msg format (b o)] .
op deletedPrng : Oid Oid -> Msg [ctor msg format (m o)] .
op prngError : Oid Oid String -> Msg [ctor msg format (r o)] .
op prngManager : -> Oid [special (...)] .
endm
There is an external object prngManager. New PRNG objects are created by sending to
it a createPrng message. As usual, the first argument of this message is the addressee (the
prngManager external object) and the second argument is the identifier of the object to which
the response will be sent. The final argument is the name of a pseudo-random number generator
algorithm to be used. Two algorithms are currently supported:5
• "MT32" - 32-bit Mersenne Twister
• "MT64" - 64-bit Mersenne Twister
After successful creation, the createdPrng message returns the identifier of a new PRNG
external object which has a name of the form prng(M ) for some natural number M . The PRNG
object receives a default seed.6 At any point the seed can be set with the setSeed/seedSet
message pair.7 Seeds should be 32-bit natural numbers for MT32 PRNGs and 64-bit for MT64
PRNGs. However, for convenience, any size seed may be given and only the appropriate low
order bits are used. Thus, an MT32 PRNG can be seeded, for example, from the number of
nanoseconds since the Unix Epoch.
The next pseudo-random number from a PRNG is obtained the PRNG object with the
getNext/gotNext pair. Once a PRNG object is no longer needed, it can be disposed of with
the deletePrng/deletedPrng message pair.
In the case of a bad algorithm or seed argument, a prngError message is returned instead
of the expected createdPrng or seedSet messages.
The following example shows how to create an MT32 PRNG object, seed it from the number
of nanoseconds since the Unix Epoch, and get a list of the first 10 pseudo-random numbers.
Note that the object-oriented syntax is used to simplify the rules but this is optional.
load prng
load time
omod PRNG-TEST is
inc PRNG .
inc TIME .
pr LIST{Nat} .
class Rlist | prng : Oid, count : Nat, numbers : List{Nat} .
ops me dummy : -> Oid .
op start : -> Configuration .
rl start
5The 32-bit and the 64-bit Mersenne Twister algorithms available in Maude are based on the algorithms
mt19937 and mt19937_64 from the C++ standard library.
6The default seed that the PRNG object receives is defined in the underlying C++ library.
7BysettingtheseedofanMT32PRNGtoN willcauseittogeneratethesequenceofnumbersthatcorrespond
to those produced by random(k) from the existing module RANDOM (see Section 8.4) for k= 0,1,2,... if Maude
is started with the flag -random-seed=N , where N is taken as 0 if no such flag is given.

<!-- manual page 306 -->
294 CHAPTER 9. EXTERNAL OBJECTS AND IO
=> <>
< me : Rlist | prng : dummy, count : 10, numbers : nil >
createPrng(prngManager, me, "MT32") .
vars M O R : Oid .
vars N S V : Nat .
var L : List{Nat} .
rl createdPrng(M, O, R)
< M : Rlist | >
=> < M : Rlist | prng : R >
getTimeSinceEpoch(timeManager, M) .
rl gotTimeSinceEpoch(M, O, S)
< M : Rlist | prng : R >
=> < M : Rlist | >
setSeed(R, M, S) .
rl seedSet(M, O)
< M : Rlist | count : s N >
=> < M : Rlist | count : N >
getNext(O, M) .
rl gotNext(M, O, V)
< M : Rlist | count : s N, numbers : L >
=> < M : Rlist | count : N, numbers : (L V) >
getNext(O, M) .
rl gotNext(M, O, V)
< M : Rlist | count : 0, numbers : L >
=> < M : Rlist | numbers : (L V) >
deletePrng(O, M) .
endom
By rewriting the term start we get the following result.
Maude> erew start .
erewrite in PRNG-TEST : start .
rewrites: 14 in 0ms cpu (0ms real) (33492 rewrites/second)
result Configuration:
<>
deletedPrng(me, prng(0))
< me : Rlist |
prng : prng(0),
count : 0,
numbers : ( 71681586 1996861547 3581246897 250383106 3459797240
1172033663 1469288671 1100035173 3869436749 4242052806) >
9.8 Control-C on external events
When a control-C is typed during regular rewriting, execution is suspended and Maude gets
into its debugger (see Chapter 21). However, when Maude is suspended on an external event,
since no internal events are possible, and it receives a control-C, it prints a message rather than
immediately aborting. A second control-C without intervening rewrites will cause an abort to
the command line without any attempt to print the current state.
The reason for this is to leave open the possibility of recovering from potential problems
happening on external objects. Following the conventions of the underlying operative systems,

<!-- manual page 307 -->
9.8. CONTROL-C ON EXTERNAL EVENTS 295
“Interruptsignalsgeneratedintheterminalaredeliveredtotheactiveprocessgroup, whichhere
includes both parent and child.” In other words, when a control-C is typed, the signal is not
only received by the Maude process, but also by all other active external processes depending
on it. Each of these processes may respond to a control-C in different ways, but there might
be cases in which we can make it recover once interrupted. If we do not want to try to recover
the execution, and it remains suspended on the same event, we may type a second control-C
to abort.
Let us illustrate some of these ideas in concrete examples. Specifically, let us revisit the
examples presented in Section 9.5. To get the execution to suspend on an external event, let
us introduce a little change in the specification of the desk calculator process example (module
PROCESS-DCinSection9.5.1). ThemodulePROCESS-DC-FAULTY(intheprocess-dc-faulty.maude
file) is exactly as the PROCESS-DC module in Section 9.5.1, but when submitting the second ex-
pression to be evaluated, instead of the message
send(IO, X, "4 * p\n")
the following message is being sent
send(IO, X, "4 * p")
Notice that without \n, the dc does not process the input, and therefore it does not respond
to our subsequent receive message. The process gets suspended waiting for an answer that
will never arrive. By typing a control-C while the process is suspended, and then a second
control-C when prompted, we observe the following behavior:
Maude> erew <>
init .
erewrite in PROCESS-MAUDE : <> init .
^C
Control-C while suspended on external event(s).
A second control-C on the same suspension or within a second will abort execution
and return to command line.
^C
Second control-C within a second.
Aborting execution and returning to command line.
Maude>
The control-C does not kill the Maude run. It shows the message
Control-C while suspended on external event(s).
A second control-C on the same suspension or within a second will abort execution
and return to command line.
indicating what the situation was when the control-C was typed, and that a second control-
C might be needed to abort. However, since an interrupt signal is sent to the dc process, it gets
killed. This is the behavior of the desktop calculator application. As we will see below, other
applications may respond differently to this kind of signals. But, since the external process
terminates, the execution is no longer suspended. Indeed, as we can see in the final output
provided, we are informed that the process exited due to the reception of a SIGINT signal.
Let us now consider our second example, the PROCESS-MAUDE specification in Section 9.5.2.
We do not change anything on this example, just make it use the PROCESS-DC specification
modified as above (available in file process-maude-faulty.maude). The question is that if we
load the specification and run it with the corresponding erew command, we observe that the
execution gets suspended. Notice that the program is creating a Maude process which loads the
process-dc-faulty.maudefile. Theexecutionoftheerewcommandgetssuspended, aswesaw

<!-- manual page 308 -->
296 CHAPTER 9. EXTERNAL OBJECTS AND IO
above. The interesting thing is that we have just learnt how Maude handles an interrupt signal,
and then, we know that if we type control-C, the external process also receives a control-C (an
interrupt signal). Upon reception of this interrupt signal, the external Maude process produces
some output, which is received by the main Maude process. It however does not receive a valid
output, which leads to another suspension, although in this case it is the Python process that
suspends, because it is unable to handle the text that it receives. If a new control-C is typed,
since it is a different suspension, the execution does not abort. Instead, we get a new message
informing us about the situation, and telling us that, if the execution remains suspended in
the same place, a second control-C will abort the execution. Since the interrupt signal received
by the Python process does not get the execution to advance, a third control-C (second on
this suspension event), forces the execution to abort. The following shows the interaction just
explained:
erewrite in PROCESS-MAUDE : <> init .
^C
Control-C while suspended on external event(s).
A second control-C on the same suspension will abort execution and return to command
line.
^C
Control-C while suspended on external event(s).
Note that this is a different suspension than the one that received a control-C 63
rewrites ago.
A second control-C on the same suspension will abort execution and return to command
line.
^C
Second control-C while suspended on external event(s).
Aborting execution and returning to command line.
Advisory: closing open files.
Maude>
Note that no final output is presented. As we have seen above, if a control-C triggers some
behavior in the external process that exits the suspension, Maude can complete its execution
to a final state or end up suspended on some other external event which again requires two
control-C events to abort. However, aborting execution is inherently a messy procedure that
never has a printable final state.

