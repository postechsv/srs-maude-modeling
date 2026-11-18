# Chapter 2: Using Maude (manual pages 31-36)

<!-- manual page 31 -->
Chapter 2
Using Maude
2.1 Getting Maude
The Maude system is available, free of charge, under the terms of the GNU General Public
License as published by the Free Software Foundation, at the Maude home page (a snapshot is
shown in Figure 2.1)
https://maude.cs.illinois.edu
There you can also find documentation about Maude, including a Maude primer, some papers
on Maude and rewriting logic, and several Maude applications, including a set of proving tools
for Maude specifications and Maude case studies.
Maude binaries are provided for selected architectures and operating systems, including
Linux and macOS. Detailed information on this can be found in the Maude web site, where
installation instructions are also available.
The complete source code of all the examples in this document is also available in the Maude
web site.
2.2 Running Maude
A Maude session can be started by calling the maude binary included in the release package in
a Linux shell window (and similarly for other platforms). For example, we can move into the
directory where the package was extracted and then invoke Maude, obtaining a banner similar
to the following, where we can see the version of the system being executed, the date it was
built, copyright information, and the current date.
~/maude-linux$ ./maude
\||||||||||||||||||/
--- Welcome to Maude ---
/||||||||||||||||||\
Maude 3.5.1 built: Jul 16 2025 12:00:00
Copyright 1997-2025 SRI International
Wed Jul 16 20:00:00 2025
Maude>
The Maude system is now ready to accept Maude modules and commands (see Chapter A
for a complete list of Maude commands). During a Maude session, the user interacts with the
system by entering a request at the Maude prompt. For example, one can quit:
19

<!-- manual page 32 -->
20 CHAPTER 2. USING MAUDE
Figure 2.1: Maude home page at maude.cs.illinois.edu

<!-- manual page 33 -->
2.2. RUNNING MAUDE 21
Maude> quit
q may be used as an abbreviation of the quit command. But please, do not leave us so
soon! One can also enter modules and use other commands. For example, we can enter the
following module SIMPLE-NAT, which specifies the natural numbers in Peano notation with a
plus operation _+_ on them.1
Maude> fmod SIMPLE-NAT is
sort Nat .
op zero : -> Nat .
op s_ : Nat -> Nat .
op _+_ : Nat Nat -> Nat .
vars N M : Nat .
eq zero + N = N .
eq s N + M = s (N + M) .
endfm
Fortunately, we do not need to write our modules in the prompt. We can input one or several
modulesbysavingtheminafileandthenenteringthefilewiththein, loadorsloadcommands
(see Appendix A.16 for details on the difference between these commands). Assuming that the
file my-nat.maude contains the module SIMPLE-NAT above, we can do the following to enter it:
Maude> load my-nat.maude
After entering the module SIMPLE-NAT into Maude, we can, for example, reduce the term
s s zero + s s s zero (which is the equivalent in Peano notation of the more usual 2 + 3)
as follows:
Maude> reduce in SIMPLE-NAT : s s zero + s s s zero .
reduce in SIMPLE-NAT : s s zero + s s s zero .
rewrites: 3 in 0ms cpu (0ms real) (~ rews/sec)
result Nat: s s s s s zero
It is not necessary to give the name of the module in which to reduce a term explicitly. All
commands that require a module refer to the current module by default, unless a module is
explicitly given. The current module is usually the last module entered or used, although we
can use the select command to select a module to be the current one (see Appendix A.15).
Maude> reduce s s zero + s s s zero .
reduce in SIMPLE-NAT : s s zero + s s s zero .
rewrites: 3 in 0ms cpu (0ms real) (~ rews/sec)
result Nat: s s s s s zero
Any action happening in the Maude system can be interrupted by typing control-C. In
particular, by hitting control-C during a reduction in progress, such reduction is interrupted
and the system gets into debugging mode (see Section 21.1.3).
Although it is not the case in the simple examples above, sometimes we get a very big term
as output from Maude. In some cases, in order to make it easier to read and understand, we
edit the presentation of the outputs given by Maude.
Whenyouexecutemaude, thefileprelude.maude, whichincludesseveralpredefinedmodules
(see Chapter 8), is automatically loaded. To find prelude.maude, the Maude interpreter checks
for it in several directories, in the following order:
1. the directories specified in the MAUDE_LIB environment variable,
2. the directory containing the executable, and
1We do not display the ‘>’ symbol that Maude adds at the beginning of each line.

<!-- manual page 34 -->
22 CHAPTER 2. USING MAUDE
3. the current directory.
It is a good idea to include the path to prelude.maude in the MAUDE_LIB environment variable
to be sure that it will always be found, because the executable finding code may not find the
directory containing the executable.
Among the predefined modules included in prelude.maude we find a module STRING that
declares sorts and operations for manipulating strings. In particular, STRING introduces the
operation _+_ to concatenate two strings. Then, to concatenate the strings “hello”, “ ”, and
“world”, you can type at the Maude prompt the following red (which is the abbreviated form
of reduce) request:
Maude> red in STRING : "hello" + " " + "world" .
reduce in STRING : "hello" + " " + "world" .
rewrites: 2 in 0ms cpu (0ms real) (~ rews/sec)
result String: "hello world"
Actually, although STRING is not the current module right after starting the system, it is
imported by the current one, CONVERSION. Thus, we can type the following, just after starting
Maude:
Maude> red "hello" + " " + "world" .
reduce in CONVERSION : "hello" + " " + "world" .
rewrites: 2 in 0ms cpu (0ms real) (~ rews/sec)
result String: "hello world"
Notice that Maude makes explicit the module in which the term is reduced, even when no
module name is given by the user.
As said above, to load for example a user-defined module HELLO-WORLD for a Maude session,
you can either type at the Maude prompt the whole module or simply type the following
in-troduce request:
Maude> in hello-world
where hello-world is a text file in the current directory containing the module HELLO-WORLD.
For files specified by a bare file name, Maude also checks for files with .maude, .fm, and
.obj extensions. Maude can also be told using the MAUDE_LIB environment variable about
other directories to use to search for files. Thus to find a file specified in the in command,
Maude searches, in order:
1. the current directory,
2. the directories in the MAUDE_LIB environment variable, and
3. the directory containing the executable.
If the desired file is in none of these places you must type its full path name.
As for user-defined modules, user requests such as the above can either be typed at the
Maude prompt or simply in-troduced with a text file containing them. In fact, many users run
Maude inside an Emacs-like editor, since this provides both text editing facilities for creating
Maude modules and saving them in files, and also UNIX shell interaction to start a Maude
session and to in-troduce during the session modules and commands created and saved in files,
as shown in Figure 2.2.
Note that text files entered in Maude can contain not only modules, but also any command.
Actually, a file can contain as many modules and commands as one wishes. When entering it
with an in or load command, Maude will read them one after another as if they were written
at the prompt of the system. Another important issue worth pointing out is that we can write
single line and multiline comments anywhere inside a module or a file. Single line comments
are started by either *** or---, and ended by the end of line. Multiline comments are started
by ***( and ended by ). Parentheses must balance within multiline comments.

<!-- manual page 35 -->
2.2. RUNNING MAUDE 23
Figure 2.2: Running Maude inside Emacs

<!-- manual page 36 -->
24 CHAPTER 2. USING MAUDE
2.3 Getting support and more information
We maintain the following mailing lists related to Maude:
• maude-users@maude.cs.illinois.edu. A moderated list for the discussion of topics of
general interest to all Maude users. This list is typically low-traffic, and contains items
such as calls for papers, announcements of new Maude related papers, and notifications
of new releases of Maude. It is important that you subscribe to this list if using Maude,
as this is the mechanism by which we will make important announcements about the
system. To subscribe, or to view the archived messages, please go to
https://lists.cs.illinois.edu/lists/info/maude-users
• maude-help@maude.cs.illinois.edu. This is an alias for submitting questions about
any aspect of the use of Maude. Messages are distributed to a group of experienced users
who have offered to provide help. This list is not open for subscription, but you can send
messages to this list at any time. Questions posted here will be automatically archived
at
https://lists.cs.illinois.edu/lists/info/maude-help
• maude-bugs@maude.cs.illinois.edu. A list for reporting any problems you experience
with Maude (see below), and also any suggestions for enhancements and improvements.
2.4 Reporting bugs in Maude
As already mentioned, bug reports should be sent to
maude-bugs@maude.cs.illinois.edu
When submitting a bug report, please include the following information:
1. Example to reproduce the bug. Ideally this should be a single file that reproduces the
bug by loading it. If your example is large and spread out in multiple files, have a file
top.maude that loads files and executes commands as necessary to reproduce the bug.
Send all the files as a tar archive, optionally compressed with gzip.
If Maude’s output is not obviously wrong (for example, an “internal error” message),
include an explanation of why the output is wrong.
If you choose to simplify the example, note that a short runtime to expose the bug is
desirable. Asmallexampletextismostlyunimportantunlessitisnecessarytounderstand
such example text in order to understand why Maude’s output is incorrect.
2. Version of Maude used. Make sure you provide information of the concrete release of
Maude (and Full Maude if it is the case). If you are not using one of the ready-made
binaries released by the Maude team, also give the versions of the compiler and tools used
to build it and the libraries linked against.
3. Platform. Include the operating system type and version number, as well as the processor
type.

