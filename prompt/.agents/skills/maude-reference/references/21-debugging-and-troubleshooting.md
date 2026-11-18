# Chapter 21: Debugging and Troubleshooting (manual pages 531-570)

<!-- manual page 531 -->
Chapter 21
Debugging and Troubleshooting
21.1 Debugging approaches
There are several approaches to debugging and optimizing Maude programs: tracing, term
coloring, using the debugger, and using the profiler.
21.1.1 Tracing
The tracing facilities allow us to follow the execution of our specifications, that is, the sequence
of rewrites or equational simplification reductions that take place. Tracing is turned on with
the command
set trace on .
A log of the trace can be captured using script or xterm logging. This can then be studied
using a text editor. Since the trace is usually voluminous, there are a number of trace options to
control just what is traced. We refer to Appendix A.8 for a complete list of tracing commands
and options.
One of the most useful options is selective tracing:
set trace select on .
trace select foo bar ([_,_]) .
This will cause only rewrites where the statement (equation, membership or rule) is labeled
with a selected name or the redex is headed by operators with a selected name to be traced.
In the above example, suppose foo and bar are rule labels, [_,_] is an operator name, and
foo is also an operator name. Then, rewrites using the rules labeled by foo or bar will be
reported, as will also rewrites with redex whose top-level operator is either foo or [_,_]. Note
that these labels or operators need not be in existence at the time the trace select command
is executed; thus it is possible to select statements and operators that will only be created at
runtime via the metalevel.
A useful option for metaprogramming is
trace exclude FOO BAR .
This will exclude the named modules from being traced and thus allows one to selectively avoid
tracing the chosen object and/or metalevel modules.
As we have mentioned, there are different commands that may help us in the control of the
trace of the execution at hand. If the number of rewrites is small, we may use the whole trace
519

<!-- manual page 532 -->
520 CHAPTER 21. DEBUGGING AND TROUBLESHOOTING
to check the behavior of our specification. However, the number of rewrites is usually big, and
considering the whole trace is completely impossible. The different options may help us, for
example, to focus on a particular rule or set of rules, exclude certain modules from the trace,
or not tracing the rewrites happening in the conditions.
Let us illustrate some of these commands to trace the bank accounts example presented in
Section 6.1. To see the trace we just need to set the trace on. After it, the trace of any rewrite
command will be given, according to the active options. By default, the application of every
equation, membership axiom, and rule will be printed, showing the corresponding substitution,
the current whole term, and the subterm on which the axiom is being applied before and after
its application. To get a flavor of the information we get, let us rewrite the bankConf term
with a bound of 1.
Maude> set trace on .
Maude> rew [1] bankConf .
rewrite [1] in BANK-ACCOUNT-TEST : bankConf .
*********** equation
eq bankConf = (((((< A-003 : Account | bal : 1250 > from A-003 to
A-002 transfer 300) debit(A-002, 400)) < A-002 : Account | bal :
250 >) debit(A-001, 150)) debit(A-001, 200)) < A-001 : Account |
bal : 300 > .
empty substitution
bankConf
--->
(((((< A-003 : Account | bal : 1250 > from A-003 to A-002 transfer
300) debit(A-002, 400)) < A-002 : Account | bal : 250 >) debit(
A-001, 150)) debit(A-001, 200)) < A-001 : Account | bal : 300 >
*********** trial #1
crl [debit] : debit(A:Oid, M:Nat) < A:Oid : Account | bal : N:Nat >
=> < A:Oid : Account | bal : (N:Nat - M:Nat) > if N:Nat >= M:Nat = true .
A:Oid --> A-001
M:Nat --> 150
N:Nat --> 300
*********** solving condition fragment
N:Nat >= M:Nat = true
*********** equation
(built-in equation for symbol _>=_)
300 >= 150
--->
true
*********** success for condition fragment
N:Nat >= M:Nat = true
A:Oid --> A-001
M:Nat --> 150
N:Nat --> 300
*********** success #1
*********** rule
crl [debit] : debit(A:Oid, M:Nat) < A:Oid : Account | bal : N:Nat > => < A:Oid
: Account | bal : (N:Nat - M:Nat) > if N:Nat >= M:Nat = true .
A:Oid --> A-001
M:Nat --> 150
N:Nat --> 300
debit(A-001, 150) debit(A-001, 200) debit(A-002, 400) < A-001 :
Account | bal : 300 > < A-002 : Account | bal : 250 > < A-003 :
Account | bal : 1250 > from A-003 to A-002 transfer 300

<!-- manual page 533 -->
21.1. DEBUGGING APPROACHES 521
--->
(debit(A-001, 200) debit(A-002, 400) < A-002 : Account | bal : 250 >
< A-003 : Account | bal : 1250 > from A-003 to A-002 transfer
300) < A-001 : Account | bal : (300 - 150) >
*********** equation
(built-in equation for symbol _-_)
300 - 150
--->
150
rewrites: 4 in 1ms cpu (1ms real) (4000 rewrites/second)
result Configuration: debit(A-001, 200) debit(A-002, 400) < A-001 :
Account | bal : 150 > < A-002 : Account | bal : 250 > < A-003 :
Account | bal : 1250 > from A-003 to A-002 transfer 300
Notice that, even though the bound for the rewrite command is one, there are four rewrites.
Recall that the bound only concerns rule application. In this trace we see how three equations—
two of them built-in—are also applied. In addition to the statement used in each rewriting step,
the trace shows the matching substitution and the whole term, before and after the application
of the statement. Notice also the information concerning the evaluation of conditions. We can
see that, although there is a match with the debit rule, this rule is not applied until the success
of its condition has been checked.
Suppose we are mainly concerned with the application of rules. In this case we may think
that there is too much “noise” due to the application of equations. We may request hiding the
information about the application of equations with the command set trace eq off. Then
the trace for rewriting the same bankConf term with the same bound of 1 is as follows:
Maude> set trace eq off .
Maude> rew [1] bankConf .
rewrite [1] in BANK-ACCOUNT-TEST : bankConf .
*********** trial #1
crl [debit] : debit(A:Oid, M:Nat) < A:Oid : Account | bal : N:Nat > => < A:Oid
: Account | bal : (N:Nat - M:Nat) > if N:Nat >= M:Nat = true .
A:Oid --> A-001
M:Nat --> 150
N:Nat --> 300
*********** solving condition fragment
N:Nat >= M:Nat = true
*********** success for condition fragment
N:Nat >= M:Nat = true
A:Oid --> A-001
M:Nat --> 150
N:Nat --> 300
*********** success #1
*********** rule
crl [debit] : debit(A:Oid, M:Nat) < A:Oid : Account | bal : N:Nat > => < A:Oid
: Account | bal : (N:Nat - M:Nat) > if N:Nat >= M:Nat = true .
A:Oid --> A-001
M:Nat --> 150
N:Nat --> 300
debit(A-001, 150) debit(A-001, 200) debit(A-002, 400) < A-001 :
Account | bal : 300 > < A-002 : Account | bal : 250 > < A-003 :
Account | bal : 1250 > from A-003 to A-002 transfer 300
--->
(debit(A-001, 200) debit(A-002, 400) < A-002 : Account | bal : 250 >

<!-- manual page 534 -->
522 CHAPTER 21. DEBUGGING AND TROUBLESHOOTING
< A-003 : Account | bal : 1250 > from A-003 to A-002 transfer
300) < A-001 : Account | bal : (300 - 150) >
rewrites: 4 in 1ms cpu (0ms real) (4000 rewrites/second)
result Configuration: debit(A-001, 200) debit(A-002, 400) < A-001 :
Account | bal : 150 > < A-002 : Account | bal : 250 > < A-003 :
Account | bal : 1250 > from A-003 to A-002 transfer 300
The selection of the concrete operator or statement label to trace may also be a good
alternative when looking for something specific. Suppose that we are suspicious of a particular
rule, say transfer. We may get the applications of such a rule for the unbounded rewrite of
the bankConf term by using the trace select command as follows.
Maude> set trace select on .
Maude> trace select transfer .
Maude> rew bankConf .
rewrite in BANK-ACCOUNT-TEST : bankConf .
*********** trial #1
crl [transfer] : < A:Oid : Account | bal : N:Nat > < B:Oid : Account | bal :
N’:Nat > fromA:Oid to B:Oid transfer M:Nat => < A:Oid : Account |
bal : (N:Nat - M:Nat) > < B:Oid : Account | bal : (M:Nat +
N’:Nat) > if N:Nat >= M:Nat = true .
A:Oid --> A-003
N:Nat --> 1250
B:Oid --> A-002
N’:Nat --> 250
M:Nat --> 300
*********** solving condition fragment
N:Nat >= M:Nat = true
*********** success for condition fragment
N:Nat >= M:Nat = true
A:Oid --> A-003
N:Nat --> 1250
B:Oid --> A-002
N’:Nat --> 250
M:Nat --> 300
*********** success #1
*********** rule
crl [transfer] < A:Oid : Account | bal : N:Nat > < B:Oid : Account | bal :
N’:Nat > from A:Oid to B:Oid transfer M:Nat => < A:Oid : Account
| bal : (N:Nat - M:Nat) > < B:Oid : Account | bal : (M:Nat +
N’:Nat) > if N:Nat >= M:Nat = true .
A:Oid --> A-003
N:Nat --> 1250
B:Oid --> A-002
N’:Nat --> 250
M:Nat --> 300
debit(A-001, 200) debit(A-002, 400) < A-001 : Account | bal : 150 >
< A-002 : Account | bal : 250 > < A-003 : Account | bal : 1250 >
from A-003 to A-002 transfer 300
--->
(debit(A-001, 200) debit(A-002, 400) < A-001 : Account | bal : 150 >)
< A-003 : Account | bal : (1250 - 300) > < A-002 : Account |
bal : (300 + 250) >
rewrites: 13 in 1ms cpu (1ms real) (13000 rewrites/second)
result Configuration: debit(A-001, 200) < A-001 : Account |

<!-- manual page 535 -->
21.1. DEBUGGING APPROACHES 523
bal : 150 > < A-002 : Account | bal : 150 > < A-003 : Account |
bal : 950 >
We may also hide some of the information being shown. For example, we may get the same
tracewithoutthesubstitutionsbeingshownwiththeset trace substitution offcommand.
Maude> set trace substitution off .
Maude> rew bankConf .
rewrite in BANK-ACCOUNT-TEST : bankConf .
*********** trial #1
crl [transfer] : < A:Oid : Account | bal : N:Nat > < B:Oid : Account | bal :
N’:Nat > from A:Oid to B:Oid transfer M:Nat => < A:Oid : Account
| bal : (N:Nat - M:Nat) > < B:Oid : Account | bal : (M:Nat +
N’:Nat) > if N:Nat >= M:Nat = true .
*********** solving condition fragment
N:Nat >= M:Nat = true
*********** success for condition fragment
N:Nat >= M:Nat = true
*********** success #1
*********** rule
crl [transfer] : < A:Oid : Account | bal : N:Nat > < B:Oid : Account | bal :
N’:Nat > from A:Oid to B:Oid transfer M:Nat => < A:Oid : Account
| bal : (N:Nat - M:Nat) > < B:Oid : Account | bal : (M:Nat +
N’:Nat) > if N:Nat >= M:Nat = true .
debit(A-001, 200) debit(A-002, 400) < A-001 : Account | bal : 150 >
< A-002 : Account | bal : 250 > < A-003 : Account | bal : 1250 >
from A-003 to A-002 transfer 300
--->
(debit(A-001, 200) debit(A-002, 400) < A-001 : Account | bal : 150 >)
< A-003 : Account | bal : (1250 - 300) > < A-002 : Account |
bal : (300 + 250) >
rewrites: 13 in 0ms cpu (0ms real) (~ rewrites/second)
result Configuration: debit(A-001, 200) < A-001 : Account |
bal : 150 > < A-002 : Account | bal : 150 > < A-003 : Account |
bal : 950 >
Let us consider now a different example, namely, the PATH module presented in Sections 3.5
and 4.3. We use it here to illustrate the trace given for membership axioms and for conditional
axioms with multiple fragments. We recall first the conditional membership axiom defining
multi-edge paths and the conditional equation defining the associativity of path concatenation.
var E : Edge .
vars P Q R S : Path .
cmb E ; P : Path if target(E) = source(P) .
ceq (P ; Q) ; R = P ; (Q ; R)
if target(P) = source(Q) /\ target(Q) = source(R) .
Nowwerequestthetraceforthereductionofthetermlength((b ; c) ; d). Theinformation
shownisparticularlyillustrativeforunderstandingthewayinwhichthemembershipaxiomsare
used and the way conditions are evaluated. Note that the equation expressing the associativity
of path concatenation has two fragments, one of which is evaluated after the other. In case the
condition of a matching equation fails another equation is attempted; furthermore, equations
with matching conditions have unbounded variables initially.
Since the full trace is more than six pages long, we use the set trace condition off
command, so that the evaluation of the conditions is omitted.

<!-- manual page 536 -->
524 CHAPTER 21. DEBUGGING AND TROUBLESHOOTING
Maude> set trace on .
Maude> set trace condition off .
Maude> red length((b ; c) ; d) .
reduce in PATH : length((b ; c) ; d) .
*********** trial #1
cmb E ; P : Path if target(E) = source(P) .
E --> b
P --> c
*********** solving condition fragment
target(E) = source(P)
*********** success for condition fragment
target(E) = source(P)
E --> b
P --> c
*********** success #1
*********** membership axiom
cmb E ; P : Path if target(E) = source(P) .
E --> b
P --> c
[Path]: b ; c becomes Path
*********** trial #2
ceq (P ; Q) ; R = P ; (Q ; R)
if target(P) = source(Q) /\ target(Q) = source(R) .
P --> b
Q --> c
R --> d
*********** solving condition fragment
target(P) = source(Q)
*********** success for condition fragment
target(P) = source(Q)
P --> b
Q --> c
R --> d
*********** solving condition fragment
target(Q) = source(R)
*********** success for condition fragment
target(Q) = source(R)
P --> b
Q --> c
R --> d
*********** success #2
*********** equation
ceq (P ; Q) ; R = P ; (Q ; R)
if target(P) = source(Q) /\ target(Q) = source(R) .
P --> b
Q --> c
R --> d
(b ; c) ; d
--->
b ; (c ; d)
*********** trial #3
cmb E ; P : Path if target(E) = source(P) .
E --> c
P --> d

<!-- manual page 537 -->
21.1. DEBUGGING APPROACHES 525
*********** solving condition fragment
target(E) = source(P)
*********** success for condition fragment
target(E) = source(P)
E --> c
P --> d
*********** success #3
*********** membership axiom
cmb E ; P : Path if target(E) = source(P) .
E --> c
P --> d
[Path]: c ; d becomes Path
*********** trial #4
cmb E ; P : Path if target(E) = source(P) .
E --> b
P --> c ; d
*********** solving condition fragment
target(E) = source(P)
*********** success for condition fragment
target(E) = source(P)
E --> b
P --> c ; d
*********** success #4
*********** membership axiom
cmb E ; P : Path if target(E) = source(P) .
E --> b
P --> c ; d
[Path]: b ; (c ; d) becomes Path
*********** trial #5
ceq length(E ; P) = 1 + length(P) if E ; P : Path .
E --> b
P --> c ; d
*********** solving condition fragment
E ; P : Path
*********** success for condition fragment
E ; P : Path
E --> b
P --> c ; d
*********** success #5
*********** equation
ceq length(E ; P) = 1 + length(P) if E ; P : Path .
E --> b
P --> c ; d
length(b ; (c ; d))
--->
1 + length(c ; d)
*********** trial #6
ceq length(E ; P) = 1 + length(P) if E ; P : Path .
E --> c
P --> d
*********** solving condition fragment
E ; P : Path
*********** success for condition fragment
E ; P : Path

<!-- manual page 538 -->
526 CHAPTER 21. DEBUGGING AND TROUBLESHOOTING
E --> c
P --> d
*********** success #6
*********** equation
ceq length(E ; P) = 1 + length(P) if E ; P : Path .
E --> c
P --> d
length(c ; d)
--->
1 + length(d)
*********** equation
eq length(E) = 1 .
E --> d
length(d)
--->
1
*********** equation
(built-in equation for symbol _+_)
1 + 1
--->
2
*********** equation
(built-in equation for symbol _+_)
1 + 2
--->
3
rewrites: 20 in 2ms cpu (1ms real) (10000 rewrites/second)
result NzNat: 3
But the trace is too long to observe what we were interested in. Suppose we just wanted
to check a possible mistake in the specification of the length function. We may select it for
filtering the equations defining it.
Maude> set trace select on .
Maude> trace select length .
Maude> red length((b ; c) ; d) .
reduce in PATH : length((b ; c) ; d) .
*********** trial #1
ceq length(E ; P) = 1 + length(P) if E ; P : Path .
E --> b
P --> c ; d
*********** solving condition fragment
E ; P : Path
*********** success for condition fragment
E ; P : Path
E --> b
P --> c ; d
*********** success #1
*********** equation
ceq length(E ; P) = 1 + length(P) if E ; P : Path .
E --> b
P --> c ; d
length(b ; (c ; d))
--->
1 + length(c ; d)

<!-- manual page 539 -->
21.1. DEBUGGING APPROACHES 527
*********** trial #2
ceq length(E ; P) = 1 + length(P) if E ; P : Path .
E --> c
P --> d
*********** solving condition fragment
E ; P : Path
*********** success for condition fragment
E ; P : Path
E --> c
P --> d
*********** success #2
*********** equation
ceq length(E ; P) = 1 + length(P) if E ; P : Path .
E --> c
P --> d
length(c ; d)
--->
1 + length(d)
*********** equation
eq length(E) = 1 .
E --> d
length(d)
--->
1
rewrites: 20 in 0ms cpu (1ms real) (~ rewrites/second)
result NzNat: 3
21.1.2 Term coloring
A common failure mode of Maude programs is when a term does not fully reduce. This is a
lack of sufficient completeness. For linear unconditional order-sorted specifications, sufficient
completeness can be checked with the SCC tool [73]. However, for general Maude specifications
proving sufficient completeness may require inductive theorem proving. If a term does not fully
reduce, that is, if nonconstructor symbols remain in the term’s canonical form, it can be difficult
to determine just where the problem began, since when a subterm fails to reduce, the enclosing
term often fails to reduce, and so on, leading to a large unreduced term. If the specification
makes consistent use of the ctor attribute, problem subterms can be pinpointed by switching
on term coloring with the command
set print color on .
Symbols within terms that are being executed (i.e., in a trace or in the final result of a
reduce command) are colored as follows:
reduced, ctor not colored
reduced, non-ctor, strangeness below blue
reduced, non-ctor, no strangeness below red
unreduced, no reduced above green
unreduced, reduced directly above magenta
unreduced, reduced not directly above cyan
If an operator is colored, this means that the term contains nonconstructors, that is, that
there is “strangeness” in the term. The different colors indicate the source of the strangeness.

<!-- manual page 540 -->
528 CHAPTER 21. DEBUGGING AND TROUBLESHOOTING
The idea is that red and magenta indicate the initial locus of a bug, while blue and cyan indicate
secondary damage. Green denotes reduction pending and cannot appear in the final result. An
example is the following module, in which there is a missing case in each of the definitions of
the _<_ and min operators (0 < 0 and min(N N), respectively).
fmod NAT-MSET-MIN is
protecting BOOL .
sorts Nat NatMSet .
subsort Nat < NatMSet .
op 0 : -> Nat [ctor] .
op s : Nat -> Nat [ctor] .
op _ _ : NatMSet NatMSet -> NatMSet [assoc comm ctor] .
op _<_ : Nat Nat -> Bool .
op min : NatMSet -> Nat .
vars N M : Nat .
var S : NatMSet .
eq 0 < s(N) = true .
eq s(N) < 0 = false .
eq s(N) < s(M) = N < M .
eq min(N N S) = min(N S) .
ceq min(N M S) = min(N S) if N < M .
ceq min(N M) = N if N < M .
eq min(N) = N .
endfm
With color printing turned on, reducing min(s(s(0)) s(s(0))) returns the term with
the min operator colored red, indicating a nonconstructor that can’t be reduced. Reducing
min(s(s(0)) min(s(0) s(0))) returns the term with the inner occurrence of the min oper-
ator colored red as above, and the outer occurrence colored blue, indicating that the problem
probably lies in a subterm.
To avoid confusion, any colors that may have been specified using the format attribute (see
Section 4.4.5) are ignored in this mode.
21.1.3 The debugger
There are three ways to get into the Maude debugger:
• a control-C interrupt during rewriting,
• prefixing a command with the keyword debug, and
• hitting a break point.
Break points are set with the command
break select foo bar ([_,_]) .
where the names refer to operators or statement (equation, membership, rule or strategy defi-
nition) labels in a way that is completely analogous to the trace select command described
in Section 21.1.1. Break points are enabled with the command
set break on .
On entering the debugger, the prompt changes to Debug(n)> where n is the debug level,
that is, the number of times the debugger has been re-entered (it is fully re-entrant). All top-
level commands can be executed from the debugger, along with four commands that are special
to the debugger:
where . Prints out the stack of pending rewrites, explaining how each one arose.

<!-- manual page 541 -->
21.1. DEBUGGING APPROACHES 529
step . Executes the next rewrite with tracing turned on.
resume . Exits the debugger and continues with the current rewriting task.
abort . Exits the debugger and abandons the current rewriting task.
We illustrate these commands using the bank accounts example presented in Section 6.4.1
(assuming it is in the file bank-account-test.maude).
We first use the debug command to activate the debugger from the begining of a rewrite.
Note the use of the where, step, and resume commands.
Maude> load bank-account-test.maude
Maude> debug rew bankConf .
rewrite in BANK-ACCOUNT-TEST : bankConf .
Debug(1)> where .
Current term is:
bankConf
which arose while executing a top level command.
Debug(1)> step .
*********** equation
eq bankConf = (((((< A-003 : Account | bal : 1250 > from A-003 to
A-002 transfer 300) debit(A-002, 400)) < A-002 : Account | bal :
250 >) debit(A-001, 150)) debit(A-001, 200)) < A-001 : Account |
bal : 300 > .
empty substitution
bankConf
--->
(((((< A-003 : Account | bal : 1250 > from A-003 to A-002 transfer
300) debit(A-002, 400)) < A-002 : Account | bal : 250 >) debit(
A-001, 150)) debit(A-001, 200)) < A-001 : Account | bal : 300 >
Debug(1)> where .
Current term is:
debit(A-001, 150) debit(A-001, 200) debit(A-002, 400) < A-001 :
Account | bal : 300 > < A-002 : Account | bal : 250 > < A-003 :
Account | bal : 1250 > from A-003 to A-002 transfer 300
which arose while executing a top level command.
Debug(1)> resume .
rewrites: 13 in 2ms cpu (85160ms real) (6500 rewrites/second)
result Configuration: debit(A-001, 200) < A-001 : Account | bal :
150 > < A-002 : Account | bal : 150 > < A-003 : Account | bal :
950 >
As said above, we can also enter into the debugger by reaching a break point or typing
control-c. In the following example we set a break point on the debit rule, take a step, and
then abort the rewrite process.
Maude> set break on .
Maude> break select debit .
Maude> rew bankConf .
rewrite in BANK-ACCOUNT-TEST : bankConf .
break on labeled rule:
crl [debit] : debit(A:Oid, M:Nat) < A:Oid : Account | bal : N:Nat > => < A:Oid
: Account | bal : (N:Nat - M:Nat) > if N:Nat >= M:Nat = true .
Debug(1)> where .
Current term is:
debit(A-001, 150) debit(A-001, 200) debit(A-002, 400) < A-001 :
Account | bal : 300 > < A-002 : Account | bal : 250 > < A-003 :
Account | bal : 1250 > from A-003 to A-002 transfer 300

<!-- manual page 542 -->
530 CHAPTER 21. DEBUGGING AND TROUBLESHOOTING
which arose while executing a top level command.
Debug(1)> step .
*********** trial #1
crl [debit] : debit(A:Oid, M:Nat) < A:Oid : Account | bal : N:Nat > => <
A:Oid : Account | bal : (N:Nat - M:Nat) > if N:Nat >= M:Nat =
true .
A:Oid --> A-001
M:Nat --> 150
N:Nat --> 300
*********** solving condition fragment
N:Nat >= M:Nat = true
Debug(1)> where .
Current term is:
300 >= 150
which arose while checking a condition during the evaluation of:
debit(A-001, 150) debit(A-001, 200) debit(A-002, 400) < A-001 :
Account | bal : 300 > < A-002 : Account | bal : 250 > < A-003 :
Account | bal : 1250 > from A-003 to A-002 transfer 300
which arose while executing a top level command.
Debug(1)> abort .
Maude>
Our last example illustrates the re-entering nature of the debugger. As said above, any
command can be used during the debugging process, allowing, for example, starting an execu-
tion while debugging another one. We execute a debug rew command, entering the debugger,
where we set a break point on the transfer rule. Notice the Debug(2)> prompt. Notice also
how after getting out of the inner debugger the break point is still active.
Maude> debug rew bankConf .
rewrite in BANK-ACCOUNT-TEST : bankConf .
Debug(1)> step .
*********** equation
eq bankConf = (((((< A-003 : Account | bal : 1250 > from A-003 to
A-002 transfer 300) debit(A-002, 400)) < A-002 : Account | bal
: 250 >) debit(A-001, 150)) debit(A-001, 200)) < A-001 : Account
| bal : 300 > .
empty substitution
bankConf
--->
(((((< A-003 : Account | bal : 1250 > from A-003 to A-002 transfer
300) debit(A-002, 400)) < A-002 : Account | bal : 250 >) debit(
A-001, 150)) debit(A-001, 200)) < A-001 : Account | bal : 300 >
Debug(1)> set break on .
Debug(1)> break select transfer .
Debug(1)> rew bankConf .
rewrite in BANK-ACCOUNT-TEST : bankConf .
break on labeled rule:
crl [transfer] : < A:Oid : Account | bal : N:Nat > < B:Oid : Account | bal :
N’:Nat > from A:Oid to B:Oid transfer M:Nat => < A:Oid : Account
| bal : (N:Nat - M:Nat) > < B:Oid : Account | bal : (M:Nat +
N’:Nat) > if N:Nat >= M:Nat = true .
Debug(2)> where .
Current term is:
debit(A-001, 200) debit(A-002, 400) < A-001 : Account | bal : 150 >
< A-002 : Account | bal : 250 > < A-003 : Account | bal : 1250 >

<!-- manual page 543 -->
21.1. DEBUGGING APPROACHES 531
from A-003 to A-002 transfer 300
which arose while executing a top level command.
Debug(2)> resume .
break on labeled rule:
crl [transfer] : < A:Oid : Account | bal : N:Nat > < B:Oid : Account | bal :
N’:Nat > from A:Oid to B:Oid transfer M:Nat => < A:Oid : Account |
bal : (N:Nat - M:Nat) > < B:Oid : Account | bal : (M:Nat + N’:Nat)
> if N:Nat >= M:Nat = true .
Debug(2)> abort .
Debug(1)> resume .
break on labeled rule:
crl [transfer] : < A:Oid : Account | bal : N:Nat > < B:Oid : Account | bal :
N’:Nat > from A:Oid to B:Oid transfer M:Nat => < A:Oid : Account
| bal : (N:Nat - M:Nat) > < B:Oid : Account | bal : (M:Nat +
N’:Nat) > if N:Nat >= M:Nat = true .
Debug(1)> set break off .
Debug(1)> resume .
rewrites: 13 in 4ms cpu (63920ms real) (2600 rewrites/second)
result Configuration: debit(A-001, 200) < A-001 : Account | bal :
150 > < A-002 : Account | bal : 150 > < A-003 : Account | bal :
950 >
21.1.4 Status report
We can find out what Maude is doing during execution without sending it a control-C (which
willclosefilesandmessupsocketcommunicationamongotherthingssinceithastoexitblocked
system calls to respond to the user).
Depending on whether Maude is running the status report may be activated either us-
ing ctrl-T or kill -INFO <pid> (Mac)1 and kill -USR1 <pid> (Linux).2 When activated,
Maude will print out the current time, rewrite counts and a summary of what it is doing on
stderr.
21.1.5 The profiler
Tuning up of specifications is something that may be useful in many practical situations. We
illustrate the use of the profiling facilities available in Maude to understand better the execution
of our specifications, helping us in this way to make them more efficient. We will discuss the
use of the profiler on two examples, namely, the specification of the Fibonacci function already
discussed in Section 4.4.8 and the specification of sorted lists presented in Section 7.4.6.
First of all, it must be clear that there is no magic recipe on how to optimize our specifi-
cations. On the contrary, although there are some guidelines that we may try to follow when
possible, it is not always the case that they work, or that they are applicable. For example, con-
ditional rules are generally expensive from a computational point of view, as are membership
axioms, but in some cases we may be interested in using proving tools for which using them
could be a better alternative. Similarly, in Section 4.4.8 we saw that using the memo attribute
was a big win in the case of the Fibonacci function, but it is not always applicable; for some
1On Mac, the status report feature uses the BSD SIGINFO extension, which is activated by control-T from
a terminal where Maude is the foreground process. It can also be activated by kill -INFO <pid>, however this
does not give the load and timing information as control-T does.
2On Linux, there is no equivalent capability so the Maude implementation follows the dd convention and
kill -USR1 <pid> can be used instead.

<!-- manual page 544 -->
532 CHAPTER 21. DEBUGGING AND TROUBLESHOOTING
specifications, the consumption of memory can become so big that we may be getting a slower
specification. There is always a tradeoff between the speedup obtained using memoization and
the amount of memory and the cost of handling it. We illustrate all these and other concerns
in this section.
Profiling is switched on by the command set profile on. When profiling is switched on, a
count of the number of executions of each statement (equation, membership, rule, and strategy
definition) is kept. For unconditional statements, the profile information is just the number of
rewrites using that statement. For conditional ones there is also the number of matches, since
not every match leads to a rewrite, due to condition failure. Moreover, when searching there
can be multiple rewrites for each match, since the condition may be solved in multiple ways.
There is a table that for each condition fragment gives:
1. the number of times the fragment was initially tested,
2. the number of times the fragment was tested due to backtracking,
3. the number of times the fragment succeeded, and
4. the number of times the fragment failed.
Normally, (1) + (2) = (3) + (4).
Special rewrites such as built-in rewrites and memoized rewrites are also tracked, but these
are associated with symbols rather than with statements. For conciseness, symbols with no
special rewrites, and statements that are not matched are omitted. There are some limitations:
metalevel rewrites are not displayed, due to the ephemeral nature of metamodules. In addition,
condition fragments associated with a match or search command are not tracked (though any
rewrites initiated by such a fragment are). If you turn profiling on or off in the debugger you
may get inconsistent results.
The profile information is associated with each module and is usually cleared at the start
of any command that can do rewrites, except continue. This behavior can be changed with
the set clear profile on / off command.
Let us first consider the Fibonacci function described in Section 4.4.8.
fmod FIBONACCI is
protecting NAT .
op fibo : Nat -> Nat .
var N : Nat .
eq fibo(0) = 0 .
eq fibo(1) = 1 .
eq fibo(s s N) = fibo(N) + fibo(s N) .
endfm
Notice in the following reductions that the times given when the profile is active are slightly
higher.
Maude> red fibo(30) .
reduce in FIBONACCI : fibo(30) .
rewrites: 4038805 in 3920ms cpu (3960ms real) (1030201 rews/sec)
result NzNat: 832040
Maude> set profile on .
Maude> red fibo(30) .
reduce in FIBONACCI : fibo(30) .
rewrites: 4038805 in 4170ms cpu (4194ms real) (968453 rews/sec)
result NzNat: 832040

<!-- manual page 545 -->
21.1. DEBUGGING APPROACHES 533
After doing the reduction with the profiler activated, we can request the collected infor-
mation by means of the command show profile. In this example, since the module has no
memberships or rules and there are no conditional axioms, the profiler gives the number of
times each of the equations has been applied and also the number of times built-in functions
are called.
Maude> show profile .
op _+_ : [Nat] [Nat] -> [Nat] .
built-in eq rewrites: 1346268 (33.3333%)
eq fibo(0) = 0 .
rewrites: 514229 (12.7322%)
eq fibo(1) = 1 .
rewrites: 832040 (20.6011%)
eq fibo(s_^2(N)) = fibo(N) + fibo(s N) .
rewrites: 1346268 (33.3333%)
In this very simple example we observe that only the three equations in the FIBONACCI
module plus the predefined addition operation on natural numbers have been used. We can
alsoobservehowtheequationsareappliedanumberoftimesrelativelysimilar, withpercentages
12, 20, and 33, respectively. More interesting is the number of times each of them is applied,
which goes to 1346268 for the third equation. Taking into account that we reduced fibo(30),
it means that the calculations have been repeated many times. As we saw in Section 4.4.8, this
is a good place to use the memo attribute: calculations on small arguments are repeated many
times and a small amount of memory is needed for storing the result of such calculations.
After adding the memo attribute to the fibo operator, we get the following results from the
profiler:
Maude> set profile on .
Maude> red fibo(30) .
reduce in FIBONACCI : fibo(30) .
rewrites: 88 in 1ms cpu (1ms real) (88000 rews/sec)
result NzNat: 832040
Maude> show profile .
op _+_ : [Nat] [Nat] -> [Nat] .
built-in eq rewrites: 29 (32.9545%)
op fibo : [Nat] -> [Nat] .
memo rewrites: 28 (31.8182%)
eq fibo(0) = 0 .
rewrites: 1 (1.13636%)
eq fibo(1) = 1 .
rewrites: 1 (1.13636%)
eq fibo(s_^2(N)) = fibo(N) + fibo(s N) .
rewrites: 29 (32.9545%)

<!-- manual page 546 -->
534 CHAPTER 21. DEBUGGING AND TROUBLESHOOTING
As we already saw in Section 4.4.8, the number of rewrites and the time consumed in the
computation have decreased dramatically. We may observe that now each of the values in the
Fibonacci sequence is calculated only once.
Let us consider now another example, namely, the parameterized module SORTED-LIST
presented in Section 7.4.6, which defines a sort SortedList{X} of sorted lists as a subsort
of the sort List{TOSET}{X} of lists. In this case we deal with a parameterized module which
imports several other modules, and which has membership axioms and equations, some of which
are conditional.
First of all, notice that by default the profiler provides information on a particular computa-
tion. In this example, it is not the same sorting a list in reverse order as an already sorted list,
and is not the same using insertion sort, mergesort, or quicksort for sorting. To have a better
insight about our specification, and thus gaining chances of improving it, we should consider
several reductions, dealing with different cases, the different sorting algorithms in our case.
To be able to run examples on big lists, with numbers initially sorted in different ways, let us
consider the following module NAT-LIST-GENERATOR, which imports the module SORTED-LIST
{NatAsToset} defining sorted lists of natural numbers, and specifies functions nats-upto, that
builds lists from zero to the specified value, and random-nats, which generates a list of the
specified number of random numbers.
fmod NAT-LIST-GENERATOR is
protecting SORTED-LIST{NatAsToset} .
protecting RANDOM .
vars N M : Nat .
op nats-upto : Nat -> NeSortedList{NatAsToset} .
eq nats-upto(s N) = nats-upto(N) ++ s N : [] .
eq nats-upto(0) = 0 : [] .
op random-nats : Nat -> List{TOSET}{NatAsToset} .
op random-nats : Nat Nat -> List{TOSET}{NatAsToset} .
eq random-nats(N) = random-nats(0, N) .
ceq random-nats(N, M)
= random(N) : random-nats(s N, M)
if N <= M .
eq random-nats(N, M) = [] [owise] .
endfm
We execute each one of the insertion-sort, mergesort, and quicksort algorithms on
three lists, namely, a sorted list, a list in reverse order, and a random one, each of them with
1000 elements.
Maude> red insertion-sort(nats-upto(1000)) .
rewrites: 2012009 in 1032ms cpu (1079ms real) (1948029 rews/sec)
result NeSortedList{NatAsToset}: 0 : 1 : 2 : 3 : 4 : 5 : 6 : 7 : ...
Maude> red insertion-sort(reverse(nats-upto(1000))) .
rewrites: 5519515 in 3634ms cpu (3694ms real) (1518667 rews/sec)
result NeSortedList{NatAsToset}: 0 : 1 : 2 : 3 : 4 : 5 : 6 : 7 : ...
Maude> red insertion-sort(random-nats(1000)) .
rewrites: 1535372 in 1286ms cpu (1397ms real) (1193166 rews/sec)
result NeSortedList{NatAsToset}: 23772 : 1738648 : 2016694 : ...

<!-- manual page 547 -->
21.1. DEBUGGING APPROACHES 535
Maude> red mergesort(nats-upto(1000)) .
rewrites: 2082358 in 1079ms cpu (1134ms real) (1928402 rews/sec)
result NeSortedList{NatAsToset}: 0 : 1 : 2 : 3 : 4 : 5 : 6 : 7 : ...
Maude> red mergesort(reverse(nats-upto(1000))) .
rewrites: 2578581 in 1210ms cpu (1221ms real) (2129622 rews/sec)
result NeSortedList{NatAsToset}: 0 : 1 : 2 : 3 : 4 : 5 : 6 : 7 : ...
Maude> red mergesort(random-nats(1000)) .
rewrites: 88005 in 71ms cpu (77ms real) (1222478 rews/sec)
result NeSortedList{NatAsToset}: 23772 : 1738648 : 2016694 : ...
Maude> red quicksort(nats-upto(1000)) .
rewrites: 6519514 in 5065ms cpu (5344ms real) (1287111 rews/sec)
result NeSortedList{NatAsToset}: 0 : 1 : 2 : 3 : 4 : 5 : 6 : 7 : ...
Maude> red quicksort(reverse(nats-upto(1000))) .
rewrites: 6528518 in 4096ms cpu (4130ms real) (1593729 rews/sec)
result NeSortedList{NatAsToset}: 0 : 1 : 2 : 3 : 4 : 5 : 6 : 7 : ...
Maude> red quicksort(random-nats(1000)) .
rewrites: 97858 in 75ms cpu (84ms real) (1287791 rews/sec)
result NeSortedList{NatAsToset}: 23772 : 1738648 : 2016694 : ...
Instead of considering the profiling information separately, we use the set clear profile
off command, so that the profiling information gets accumulated.
Maude> set clear profile off .
Maude> set profile on .
Maude> red insertion-sort(nats-upto(1000)) .
rewrites: 2012009 in 1169ms cpu (1351ms real) (1719927 rews/sec)
result NeSortedList{NatAsToset}: 0 : 1 : 2 : 3 : 4 : 5 : 6 : 7 : ...
Maude> red insertion-sort(reverse(nats-upto(1000))) .
...
As mentioned above, with the profiler active, the times taken by the reductions are slightly
higher. The number of rewrites and cpu time for each of the cases is presented in the tables
displayed in Figure 21.1 (page 540), where we also include these values for all the different
executions discussed in the rest of the section.3 The information shown by the profiler with the
show profile command is three pages long; we just comment the most interesting pieces.
1. Predefined operators _+_, _quo_, _<=_, _>_, and random are used an important number
of times, in particular _<=_ and _>_, which are applied, respectively, 7599823 (28.2%) and
1770593 times (6.6%). Notice that _<=_ is only used in the conditions of one of the mem-
bership axioms and in some of the conditions of the equations for insert-list, merge,
leq-elems, gr-elems, and random-nats; the operator _>_ is used in the conditions of
the equations for insert-list, mergesort, merge, leq-elems, and gr-elems.
op _+_ : [Nat] [Nat] -> [Nat] .
built-in eq rewrites: 29961 (0.111124%)
3All these figures have been obtained running Maude during a hot summer night on a Linux platform with
an Intel Pentium M760 2GHz processor and 1GB of memory.

<!-- manual page 548 -->
536 CHAPTER 21. DEBUGGING AND TROUBLESHOOTING
op _quo_ : [Nat] [Nat] -> [Nat] .
built-in eq rewrites: 3000 (0.0111269%)
op _<=_ : [Nat] [Nat] -> [Bool] .
built-in eq rewrites: 7599823 (28.1874%)
op _>_ : [Nat] [Nat] -> [Bool] .
built-in eq rewrites: 1770593 (6.56706%)
op random : [Nat] -> [Nat] .
built-in eq rewrites: 3003 (0.011138%)
2. From the numbers for the membership axioms we may conclude that they are applied
a considerable number of times, in particular the conditional one—4790561 rewrites
(17.8%)—. Note that for conditional membership axioms, as for all conditional axioms,
the system gives information on the number of matches, that is, the number of times that
the conditions are reduced. It also provides the number of times each one of the fragments
of the condition is reduced. In the case of the membership axioms in this specification,
there is only one fragment. The part of the output corresponding to the membership
axioms is the following:
mb [] : SortedList{NatAsToset} .
rewrites: 22040 (0.0817455%)
mb N : [] : NeSortedList{NatAsToset} .
rewrites: 17505 (0.0649254%)
cmb N : NEOL:NeSortedList{NatAsToset} : NeSortedList{NatAsToset}
if N <= head(NEOL:NeSortedList{NatAsToset}) = true .
lhs matches: 4795972 rewrites: 4790561 (17.768%)
Fragment Initial tries Resolve tries Successes Failures
1 4795972 0 4790561 5411
We see that the condition has been checked 4795972 times, out of which only 5411 failed.
3. From the equations specifying the insertion-sort algorithm, the ones used more times
are the two conditional ones for the insert-list function. From the information in
the profile we see that these conditional equations have been attempted almost the same
number of times, 756888 and 754895. We also see that both have been applied almost
the same number of times, because the one that was attempted first almost always failed
the evaluation of its condition, and then the second equation was applied.
eq insertion-sort([]) = [] .
rewrites: 3 (1.11269e-05%)
eq insertion-sort(N : L:List{TOSET}{NatAsToset}) = insert-list(
insertion-sort(L:List{TOSET}{NatAsToset}), N) .
rewrites: 3003 (0.011138%)
eq insert-list([], M) = M : [] .
rewrites: 1010 (0.00374605%)
ceq insert-list(N : OL:SortedList{NatAsToset}, M) = M : N :
OL:SortedList{NatAsToset} if M <= N = true .
lhs matches: 756888 rewrites: 1993 (0.00739196%)

<!-- manual page 549 -->
21.1. DEBUGGING APPROACHES 537
Fragment Initial tries Resolve tries Successes Failures
1 756888 0 1993 754895
ceq insert-list(N : OL:SortedList{NatAsToset}, M) = N :
insert-list(OL:SortedList{NatAsToset}, M) if M > N = true .
lhs matches: 754895 rewrites: 754895 (2.79988%)
Fragment Initial tries Resolve tries Successes Failures
1 754895 0 754895 0
4. The information on the equations for mergesort presents a similar pattern, the main
difference being that there is only one conditional equation, since merge is declared as
commutative. In this case the number of rewrites of all the equations is relatively small,
much smaller than the total of rewrites in the computation. This makes us think that
the weight of the computation of the memberships and generation of the lists to sort is
much higher that the sorting itself.
eq mergesort(N : []) = N : [] .
rewrites: 3003 (0.011138%)
ceq mergesort(L:List{TOSET}{NatAsToset}) = merge(mergesort(take
length(L:List{TOSET}{NatAsToset}) quo 2 from L:List{TOSET}{
NatAsToset}),mergesort(throw length(L:List{TOSET}{
NatAsToset}) quo 2 from L:List{TOSET}{NatAsToset})) if
length(L:List{TOSET}{NatAsToset}) > 1 = true .
lhs matches: 3000 rewrites: 3000 (0.0111269%)
Fragment Initial tries Resolve tries Successes Failures
1 3000 0 3000 0
eq merge([], OL:SortedList{NatAsToset}) = OL:SortedList{
NatAsToset} .
rewrites: 3000 (0.0111269%)
ceq merge(N : OL:SortedList{NatAsToset}, M : OL’:SortedList{
NatAsToset}) = N :merge(OL:SortedList{NatAsToset}, M :
OL’:SortedList{NatAsToset}) if N <= M = true .
lhs matches: 18705 rewrites: 18705 (0.0693761%)
Fragment Initial tries Resolve tries Successes Failures
1 18705 0 18705 0
5. The information for the quicksort algorithm follows a similar pattern as well. However,
in this case it is interesting to notice that the equations for the leq-elems and gr-elems
operations are attempted the same number of times, and for each of these operations the
condition (say, N <= M) of one of the equations fails in around half of the cases, being
then used the other equation (with condition, say, N > M.
eq quicksort([]) = [] .
rewrites: 3006 (0.0111491%)
eq quicksort(N : L:List{TOSET}{NatAsToset}) = quicksort(
leq-elems(L:List{TOSET}{NatAsToset}, N)) ++ N : quicksort(
gr-elems(L:List{TOSET}{NatAsToset}, N)) .
rewrites: 3003 (0.011138%)
eq leq-elems([], M) = [] .
rewrites: 3003 (0.011138%)

<!-- manual page 550 -->
538 CHAPTER 21. DEBUGGING AND TROUBLESHOOTING
ceq leq-elems(N : L:List{TOSET}{NatAsToset}, M) = N :
leq-elems(L:List{TOSET}{NatAsToset}, M) if N <= M = true .
lhs matches: 1012626 rewrites: 506277 (1.87776%)
Fragment Initial tries Resolve tries Successes Failures
1 1012626 0 506277 506349
ceq leq-elems(N : L:List{TOSET}{NatAsToset}, M) = leq-elems(
L:List{TOSET}{NatAsToset}, M) if N > M = true .
lhs matches: 506349 rewrites: 506349 (1.87803%)
Fragment Initial tries Resolve tries Successes Failures
1 506349 0 506349 0
eq gr-elems([], M) = [] .
rewrites: 3003 (0.011138%)
ceq gr-elems(N : L:List{TOSET}{NatAsToset}, M) = gr-elems(
L:List{TOSET}{NatAsToset}, M) if N <= M = true .
lhs matches: 1012626 rewrites: 506277 (1.87776%)
Fragment Initial tries Resolve tries Successes Failures
1 1012626 0 506277 506349
ceq gr-elems(N : L:List{TOSET}{NatAsToset}, M) = N : gr-elems(
L:List{TOSET}{NatAsToset}, M) if N > M = true .
lhs matches: 506349 rewrites: 506349 (1.87803%)
Fragment Initial tries Resolve tries Successes Failures
1 506349 0 506349 0
6. From the rest of the equations applied we may highlight those for the head and _++_
operations.
eq head(E:Nat : L:List{TOSET}{NatAsToset}) = E:Nat .
rewrites: 4795972 (17.7881%)
eq [] ++ L:List{TOSET}{NatAsToset} = L:List{TOSET}{
NatAsToset} .
rewrites: 12006 (0.0445298%)
eq (E:Nat : L:List{TOSET}{NatAsToset}) ++ L’:List{TOSET}{
NatAsToset} =E:Nat : (L:List{TOSET}{NatAsToset} ++ L’:List{
Toset}{NatAsToset}) .
rewrites: 5010777 (18.5848%)
The equation for the head function is used in the evaluation of the condition of the
membership axiom. The concatenation operator is used in the quicksort, nats-upto,
and reverse functions.
Taking all the information provided by the profiler into account, we may think of doing
different types of modifications to the original specification.
• None of the operators seems to be appropriate for memoization, since they are used on
many different arguments, and if repeated, the size of the argument lists is so big that it
is probably not worthy storing the results.
Let us, in any case, add the memo attribute, e.g., to the head operator; the result for one
of the reductions above is the following:
Maude> red insertion-sort(random-nats(1000)) .

<!-- manual page 551 -->
21.1. DEBUGGING APPROACHES 539
rewrites: 1535372 in 18500ms cpu (19221ms real) (82992 rews/sec)
result NeSortedList{NatAsToset}: 23772 : 1738648 : 2016694 : ...
That is, it goes from 1286 milliseconds cpu time to 18500. See Figure 21.1 for the rest of
the values.
• Conditional equations are in general computationally expensive. Let us write the two
conditional equations for insertion-sort as one single unconditional equation:
eq insert-list(N : OL, M)
= if M <= N
then M : N : OL
else N : insert-list(OL, M)
fi .
The result for the same reduction is the following:
Maude> reduce insertion-sort(random-nats(1000)) .
rewrites: 1536365 in 638ms cpu (704ms real) (2404692 rews/sec)
result NeSortedList{NatAsToset}: 23772 : 1738648 : 2016694 : ...
Although the number of rewrites increases slightly—from 1535372 to 1536365—, the
amount of cpu time has dropped to a half—from 1286 ms. to 638 ms.
• We may use the owise attribute for making the conditional equation of the mergesort
function unconditional, that is, we may write it as:
eq mergesort(L)
= merge(mergesort(take (length(L) quo 2) from L),
mergesort(throw (length(L) quo 2) from L))
[owise] .
Although this clearly improves the equation, the win is not very significant. To test it,
we run the mergesort algorithm on a randomly generated list.
Maude> red mergesort(random-nats(1000)) .
rewrites: 87005 in 68ms cpu (110ms real) (1261124 rews/sec)
result NeSortedList{NatAsToset}: 237728 : 17386481 : ...
The number of rewrites goes from 88005 to 87005 and the cpu time goes from 71 ms. to
68 ms.
• A more important gain may be obtained by improving the splitting of the lists for the
quicksort algorithm. Let us join the leq-elems and gr-elems functions in one single
leq-gr-elems returning a pair of lists, one with the smaller or equal elements and the
other with the greater ones.
op quicksort : List{TOSET}{X} -> SortedList{X} .
op leq-gr-elems : List{TOSET}{X} X$Elt -> LGResult .
op leq-gr-elems : List{TOSET}{X} List{TOSET}{X} List{TOSET}{X}
X$Elt -> LGResult .
sort LGResult .
op {_,_} : List{TOSET}{X} List{TOSET}{X} -> LGResult .
eq quicksort([]) = [] .
ceq quicksort(N : L)
= quicksort(L’) ++ (N : quicksort(L’’))
if {L’, L’’} := leq-gr-elems(L, N).

<!-- manual page 552 -->
540 CHAPTER 21. DEBUGGING AND TROUBLESHOOTING
original spec. memo head unconditional
insert-list
profiler on profiler off
rews 2012009 2013009
nats-upto
ms 1169 1032 96188 1029
rews 5519515 5519515
reverse-nats-upto
ms 4189 3634 193977 2376
rews 1535372 1536365
random-nats
ms 1434 1286 18500 638
insertion-sort
original spec. memo head merge owise
profiler on profiler off
rews 2082358 2081358
nats-upto
ms 1223 1079 95760 1079
rews 2578581 2577581
reverse-nats-upto
ms 1409 1210 99579 1204
rews 88005 87005
random-nats
ms 108 71 559 68
mergesort
original spec. memo head improved
splitting
profiler on profiler off
rews 6519514 5267013
nats-upto
ms 5671 5065 221387 2523
rews 6528518 5777017
reverse-nats-upto
ms 4451 4096 196991 2650
rews 97858 69612
random-nats
ms 83 75 693 31
quicksort
Figure 21.1: Number of rewrites and CPU time for different versions of the sorting algorithms

<!-- manual page 553 -->
21.2. DEBUGGING STRATEGY EXECUTIONS 541
eq leq-gr-elems(L, M) = leq-gr-elems(L, [], [], M) .
eq leq-gr-elems([], L, L’, M) = {L, L’} .
eq leq-gr-elems(N : L, L’, L’’, M)
= if N <= M
then leq-gr-elems(L, N : L’, L’’, M)
else leq-gr-elems(L, L’, N : L’’, M)
fi .
The execution of the quicksort function on a list of randomly generated numbers takes
now 69612 rewrites (against 97858) and does the reduction in 31 milliseconds (around 75
before).
Maude> red quicksort(random-nats(1000)) .
reduce in SORTED-LIST-TEST : quicksort(random-nats(1000)) .
rewrites: 69612 in 31ms cpu (72ms real) (2175714 rews/sec)
result NeSortedList{NatAsToset}: 237728 : 17386481 : ...
There is still room for improvement in this specification. For instance, some operations
on lists can be made more efficient by means of tail-recursive definitions with accumulator
arguments, in the style of the definitions shown in Section 8.14.1.
21.1.6 Performance note
Turning on tracing, break points, or profiling causes Maude to run much more slowly, because
these options force execution through a slow path that performs extensive bookkeeping before
and after each rewrite, membership application, and condition fragment check. Therefore,
execution will be significantly slower if one or more of these options is on, even if no tracing
information is output, no break points are encountered, and the profile information is never
examined.
To ensure Maude is running at full speed one must use:
set trace off .
set break off .
set profile off .
21.2 Debugging strategy executions
The approaches described in Section 21.1 can also be applied for debugging the execution of
strategies. Whenusingthesrewriteanddsrewritecommands, wecanexaminetheequations,
membership axioms, and rules that are executed as part of the strategy-controlled rewriting.
Equations and membership axioms do not only occur in the evaluation of rules and rule condi-
tions, but they can also appear in strategy-specific contexts like the evaluation of the conditions
of the strategy operators, or the reduction of strategy call arguments and substitution values.
Moreover, we can also trace, stop, and profile each time a strategy definition is used to execute
a strategy call.
However, debugging strategies has an additional hardness. Since the evaluation of strategies
is a search for solutions in a restrained rewriting subtree (see Section 10.4), several execution
paths may be being explored at the same time, and their traces will appear interleaved in the
output. This affect srewrite more than dsrewrite.
Let us illustrate this with the 8-queens example in Section 10.3. Tracing of strategy defini-
tions is enabled by default, although it can be switched with the command
set trace sd on/off .

<!-- manual page 554 -->
542 CHAPTER 21. DEBUGGING AND TROUBLESHOOTING
The trace of a strategy call includes the definition being applied, the call term (except for
strategies without arguments), and the matching substitution for the lefthand side and the
condition. The subject term being rewritten can also be shown by setting
set trace whole on .
In the example, we will disable tracing for any other statement, and execute the srewrite
command using the solve strategy, which recursively applies itself until a solution to the 8-
queens problem is found.
Maude> set trace mb off . set trace eq off . set trace rl off . set trace whole on .
Maude> srew [1] 1 2 using solve .
*********** strategy call
sd solve := ...
subject --> 1 2
empty substitution
*********** strategy call
sd expand := ...
subject --> 1 2
empty substitution
*********** strategy call
sd solve := ...
subject --> 1 2 4
empty substitution
*********** strategy call
sd solve := ...
subject --> 1 2 5
empty substitution
*********** strategy call
sd solve := ...
subject --> 1 2 6
empty substitution
*********** strategy call
sd solve := ...
subject --> 1 2 7
empty substitution
*********** strategy call
sd expand := ...
subject --> 1 2 4
empty substitution
...
Solution 1
rewrites: 9793 in 139ms cpu (139ms real) (70093 rewrites/second)
result NeList{Nat}: 1 2 4 6 8 3 5 7
The complete trace records 140 strategy calls, but this fragment is enough to observe that
different search paths are interleaved in the output. The printed solution 1 2 4 6 8 3 5 7
is being explored at the same time as other potential solutions. Indeed, the 1 2 4 board is
not expanded with a call to expand until solve has been applied to a few other continuations
of 1 2. As we have said before, the dsrewrite command provides clearer traces, since the
execution tree is explored in depth. However, to obtain the path to a given solution, we still
have to discard failed branches that may have appeared during the search. In this case, only
17 strategy calls are listed before the solution is found.

<!-- manual page 555 -->
21.2. DEBUGGING STRATEGY EXECUTIONS 543
Maude> dsrew [1] 1 2 using solve .
*********** strategy call
sd solve := ...
subject --> 1 2
empty substitution
*********** strategy call
sd expand := ...
subject --> 1 2
empty substitution
*********** strategy call
sd solve := ...
subject --> 1 2 4
empty substitution
*********** strategy call
sd expand := ...
subject --> 1 2 4
empty substitution
...
*********** strategy call
sd solve := ...
subject --> 1 2 4 6 3 5
empty substitution
*********** strategy call
sd expand := ...
subject --> 1 2 4 6 3 5
empty substitution
...
*********** strategy call
sd solve := ...
subject --> 1 2 4 6 8 3 5 7
empty substitution
Solution 1
rewrites: 891
result NeList{Nat}: 1 2 4 6 8 3 5 7
Tracing strategy calls is more instructive when they have parameters or when they are
conditional, since we can see the matching substitution and the condition trials. For instance,
we can trace the moveAll strategy of the Hanoi tower puzzle in Section 10.2. In order to limit
tracing to one of the definitions of moveAll, we put a label moveAll on it.
sd moveAll(S, T, s(C)) := moveAll(S, third(S, T), C) ;
move[S <- S, T <- T] ;
moveAll(third(S, T), T, C) [label moveAll] .
Then, we can see the matching substitution for the lefthand side of this moveAll definition,
and learn how the disks are moved to solve the puzzle.
Maude> set trace select on . trace select moveAll .
Maude> srew (0)[3 2 1] (1)[nil] (2)[nil] using moveAll(0, 2, 3) .
*********** strategy call
sd moveAll(S, T, s C) := ... [label moveAll] .
call term --> moveAll(0, 2, 3)
S --> 0
T --> 2
C --> 2

<!-- manual page 556 -->
544 CHAPTER 21. DEBUGGING AND TROUBLESHOOTING
*********** strategy call
sd moveAll(S, T, s C) := ... [label moveAll] .
call term --> moveAll(0, 1, 2)
S --> 0
T --> 1
C --> 1
*********** strategy call
sd moveAll(S, T, s C) := ... [label moveAll] .
call term --> moveAll(0, 2, 1)
S --> 0
T --> 2
C --> 0
*********** strategy call
sd moveAll(S, T, s C) := ... [label moveAll] .
call term --> moveAll(2, 1, 1)
S --> 2
T --> 1
C --> 0
*********** strategy call
sd moveAll(S, T, s C) := ... [label moveAll] .
call term --> moveAll(1, 2, 2)
S --> 1
T --> 2
C --> 1
*********** strategy call
sd moveAll(S, T, s C) := ... [label moveAll] .
call term --> moveAll(1, 0, 1)
S --> 1
T --> 0
C --> 0
*********** strategy call
sd moveAll(S, T, s C) := ... [label moveAll] .
call term --> moveAll(0, 2, 1)
S --> 0
T --> 2
C --> 0
Solution 1
rewrites: 24
result Hanoi: (0)[nil] (1)[nil] (2)[3 2 1]
No more solutions.
rewrites: 24
Debugging is also available in the strategy-related commands: srewrite and dsrewrite
can be invoked with the debug keyword prefix to enter the debugger, and break points can be
set like for other statements. For instance, we could follow the clause walk of the previous trace
using the debugger.
Maude> set break on . break select moveAll .
Maude> srew (0)[3 2 1] (1)[nil] (2)[nil] using moveAll(0, 2, 3) .
break on labeled strategy definition:
sd moveAll(S, T, s C) := ... [label moveAll] .
Debug(1)> where .
Current term is:

<!-- manual page 557 -->
21.2. DEBUGGING STRATEGY EXECUTIONS 545
(0)[3 2 1] (1)[nil] (2)[nil]
which arose while executing a top level command.
Debug(1)> resume .
break on labeled strategy definition:
sd moveAll(S, T, s C) := ... [label moveAll] .
Debug(1)> abort .
Setting artificial break points (or trace points) at any position of a complex strategy expres-
sion is possible and could be useful when debugging strategies. Suppose we have the strategy
(r1 ; r2) ? r1 : r2
and we want to know what it the subject term just after r1 and before r2 in the condition. We
can declare a strategy bp1 and define it as
sd bp1 := idle [label bp1] .
Then, we evaluate the strategy (r1 ; bp1 ; r2) ? r1 : r2 breaking or tracing at bp1.
Conditional breakpoint can even be established, depending on the variables from the context
or on the subject term, by means of a conditional strategy with arguments invoked like
(r1 ; matchrew S by S using bp(S, V) ; r2) ? r1 : r2
for some variable S of the subject sort, and some variables V from the context.
Finally, strategy commands can be profiled too. As an example, we include the profile for
the execution of the moveAll strategy in the last example.
Maude> set profile on .
Maude> srew (0)[3 2 1] (1)[nil] (2)[nil] using moveAll(0, 2, 3) .
Solution 1
rewrites: 24
result Hanoi: (0)[nil] (1)[nil] (2)[3 2 1]
No more solutions.
rewrites: 24
Maude> show profile .
op _>_ : [NatList,Set{Nat}] [NatList,Set{Nat}] -> [Bool] .
built-in eq rewrites: 3 (7.69231%)
ceq third(N:Nat, M) = K:Nat if N:Nat, M, K:Nat := 0, 1, 2 .
lhs matches: 14 rewrites: 14 (35.8974%)
Fragment Initial tries Resolve tries Successes Failures
1 14 0 14 0
crl (S)[L1:NatList D1:Nat] (T)[L2:NatList D2:Nat] => (S)[L1:NatList] (T)[
L2:NatList D2:Nat D1:Nat] if D2:Nat > D1:Nat = true [label move] .
rewrites: 3 (7.69231%)
rl (S)[L1:NatList D1:Nat] (T)[nil] => (S)[L1:NatList] (T)[D1:Nat] [label
move] .
rewrites: 4 (10.2564%)
sd moveAll(S, T, 0) := idle .
rewrites: 8 (20.5128%)
sd moveAll(S, T, s C) := moveAll(S, third(S, T), C) ; move[S <- S, T <- T]

<!-- manual page 558 -->
546 CHAPTER 21. DEBUGGING AND TROUBLESHOOTING
; moveAll(third(S, T), T, C) [label moveAll] .
rewrites: 7 (17.9487%)
21.3 Traps and known problems
We list some commonly encountered problems with Maude.
21.3.1 Associativity and idempotency
Remember that the attributes assoc and idem (see Section 4.4.1) cannot be used together in
any combination of attributes, because the appropriate matching and normalization algorithms
have not been developed yet.
This requirement is quietly enforced by ignoring the attribute idem where necessary.
Let us consider the following example, in which we wrongly declare an operator with the
attributes assoc and idem appearing together.
fmod WRONG-NAT-SET is
pr NAT .
sort WNatSet .
subsort Nat < WNatSet .
op none : -> WNatSet [ctor] .
op __ : WNatSet WNatSet -> WNatSet
[ctor assoc comm idem id: none] .
endfm
When we reduce a term like, e.g., 4 4 5 2, the duplication does not disappear, because
Maude has ignored the idempotency attribute; the remaining attributes are applied as usual.
Maude> red 4 4 5 2 .
result WNatSet: 2 4 4 5
We can solve this by adding explicitly an idempotency equation, as we have seen, for exam-
ple, in Section 8.14.2.
Combining idem with attributes other than assoc is all right. For example, the following
module combines idempotency with commutativity.
fmod COMM-IDEM-EX is
pr NAT .
sort CI .
subsort Nat < CI .
op f : CI CI -> CI [ctor comm idem] .
vars N M : Nat .
var C : CI .
op g : CI -> Nat .
eq g(f(N, M)) = 0 .
eq g(C) = 1 [owise] .
endfm
Maude> red f(2, 2) .
result NzNat: 2
Maude> red f(2, 3) == f(3, 2) .
result Bool: true

<!-- manual page 559 -->
21.3. TRAPS AND KNOWN PROBLEMS 547
Notice that in this module, because of matching modulo commutativity and idempotency,
the first equation for g can be applied to a term such as, e.g., g(2). For example, we have the
following reductions:
Maude> red g(2) .
result Zero: 0
Maude> red g(f(2, f(2, 2))) .
result Zero: 0
Maude> red g(f(2, f(3, 4))) .
result NzNat: 1
21.3.2 Internal errors and stack overflow
There are several situations where the Maude interpreter may abruptly exit.
Internal errors
Bugs in the Maude interpreter that cause illegal memory addresses to be dereferenced typically
cause a hardware segmentation violation, which, in the release build, is caught and Maude exits
with a message:
Maude internal error.
Please submit a bug report to: maude-bugs@lists.cs.illinois.edu
Please include the platform details, Maude version, and a file
‘crash.maude’ that can be loaded to reproduce the crash (it may load
other files). Do not bother trying to simplify your example unless the
runtime to the bug being visible is greater than 10 seconds.
Such an error can also be caused by using one version of Maude with a file containing hooks,
such a prelude.maude, that came with a different version of Maude. Since hooks can change
between versions, they are not rigorously checked as they are only intended for developer use.
While Maude’s self-consistency checks are normally compiled out of the release build for
performance reasons, a few low cost checks are left in at critical points. If one of these fails,
Maude exits with a description of the self-check that failed and the following message:
Maude self-check failure.
Please submit a bug report to: maude-bugs@lists.cs.illinois.edu
Please include the platform details, Maude version, and a file
’crash.maude’ that can be loaded to reproduce the crash (it may load
other files). Do not bother trying to simplify your example unless the
runtime to the bug being visible is greater than 10 seconds.
Stack overflow
The Maude interpreter uses C++ recursion, and hence the system stack, for recursive traversal
of terms, recursive calls to evaluate conditions on statements and recursive calls to explore the
states in a finite model among other things. If the recursion becomes very deep, the stack can
grow past the bounds set by the operating system, which often defaults to 8192K. In this case
Maude exits with the following message:
Fatal error: stack overflow.
This can happen because you have an infinite computation, say a runaway
recursion, or are model checking an infinite model. It can also happen because

<!-- manual page 560 -->
548 CHAPTER 21. DEBUGGING AND TROUBLESHOOTING
the stacksize limit in your environment is set too low for the computation
you are trying to do. You can find the value of your stacksize with the
tcsh command ’limit stacksize’ or the bash command ’ulimit -s’.
Depending on your operating system configuration you may be able to
increase your stacksize with the tcsh command ’unlimit stacksize’
or the bash command ’ulimit -s unlimited’.
21.3.3 Bare variable lefthand sides
The use of a bare variable lefthand side for an equation, rule, or membership axiom may lead
to unexpected nontermination. The recommended place to use them is in statements declared
with the nonexec attribute, which are only going to be applied via a strategy language. Using
them in membership axioms is seductive, but very tricky. For example:
subsort Prime < Nat .
var N : Nat .
cmb N : Prime if favoritePrimeTest(N) .
willendupwiththemembershipaxiomandfavoritePrimeTestbeingappliedtoeveryreduced
term of sort Nat, including those that arise during evaluation of favoritePrimeTest(N) with
likely nontermination.
21.3.4 Operator overloading and associativity
The situation where two ad-hoc overloaded operators have the same kinds in their arities
but different ones in their coarities causes a warning to be emitted, as already mentioned in
Section 3.6. For example, loading the file overloading-assoc-warning.maude containing the
module
fmod OVER-ASSOC-EX1 is
sorts Foo Bar .
op f : Foo -> Foo .
op f : Foo -> Bar .
endfm
causes the following warning:
Warning: "overloading-assoc-warning.maude", line 8 (fmod
OVER-ASSOC-EX1): declaration for f has the same domain kinds as
the declaration on "overloading-assoc-warning.maude", line 7
(fmod OVER-ASSOC-EX1) but a different range kind.
A similar warning is obtained in the case where the arities differ but might look the same
because of associativity, like in the following example (loaded as before):
fmod OVER-ASSOC-EX2 is
sort Foo .
op f : Foo Foo -> Foo [assoc] .
op f : Foo Foo Foo -> Foo .
endfm
Warning: "overloading-assoc-warning.maude", line 22
(fmod OVER-ASSOC-EX2): declaration for f clashes with
declaration on "overloading-assoc-warning.maude", line 21 (fmod
OVER-ASSOC-EX2) because of associativity.

<!-- manual page 561 -->
21.3. TRAPS AND KNOWN PROBLEMS 549
21.3.5 Preregularity and equational attributes
We recall from Section 3.8 that Maude assumes that modules are preregular and generates
warnings when a module contains operator declarations that do not satisfy this property. This
meansthatforeachpossiblecombinationofargumentsortstheresultingtermhasauniqueleast
type, which is usually a sort but might also be the kind, depending on the operator declarations.
However,asalsoexplainedinSection3.8,inthepresenceofequationalattributes,suchasassoc,
comm, id:, and idem (see Section 4.4.1), preregularity must be understood modulo the axioms
Adeclared by such attributes. That is, we want not just each term t, but also each equivalence
class [t]A to have a least sort. Therefore, there is an additional requirement for an operator
that is declared associative, namely, that the least type of a term should not depend on the
way nested operators are associated. Let us explain this situation in some detail.
Theassocattribute, statingthatabinaryoperatorisassociative, appearsusuallyassociated
with declarations of operators whose arguments are both of the same sort, like, for example,
op _+_ : Nat Nat -> Nat [assoc] .
However, inthepresenceofsubsortsandoverloadedoperatorsitalsomakessensetohavebinary
operators whose arguments are not the same, but are related via subsorting; for example, to
make it explicit that the addition of a natural number to a nonzero natural number produces
a nonzero natural number, we can have an additional declaration
op _+_ : NzNat Nat -> NzNat [assoc] .
or also (see Section 4.4.6)
op _+_ : NzNat Nat -> NzNat [ditto] .
Thus, in general, the assoc attribute is allowed for binary operators such that the two
argument sorts and the result sort all belong to the same connected component. Therefore, it
is possible to consider a module like the following:
fmod NON-ASSOCIATIVE-EX is
sorts s1 s2 .
subsort s1 < s2 .
op f : s1 s2 -> s2 [assoc] .
op a : -> s1 .
eq f(a,a) = a .
endfm
If we try to reduce the term f(a,a), we get the following warning:
Maude> red f(a, a) .
Warning: sort declarations for associative operator f are
non-associative on 2 out of 27 sort triples. First such triple is
(s1, s1, s2).
reduce in NON-ASSOCIATIVE-EX : f(a, a) .
rewrites: 1 in 0ms cpu (0ms real) (~ rews/sec)
result s1: a
Maude has checked the preregularity property on the associative operator f. It is enough
to check this property on each triple of types, and when it fails to hold Maude returns the first
such triple. In this example we have three possible types for each one of the two arguments and
also for the result, namely, the sorts s1 and s2, and the corresponding kind [s2], and therefore
we have 33 = 27 possible triples. Among those, the triple (s1, s1, s2) does not satisfy the
preregularity checking, because f(X:s1, X:s1) has sort s2, f(X:s1, X:s2) has sort s2, and
f(X:s2, X:s2) has kind [s2], but no sort; thus the flattened term f(X:s1, X:s1, X:s2)

<!-- manual page 562 -->
550 CHAPTER 21. DEBUGGING AND TROUBLESHOOTING
could have either sort s2, by grouping the arguments as f(X:s1, f(X:s1, X:s2)), or kind
[s2], by grouping the arguments as f(f(X:s1, X:s1), X:s2)). To sum up, the sort structure
for the operator f is said to be non-associative on the triple (s1, s1, s2).
Two ways of avoiding this undesirable situation are the following: either having a unique
declaration at the top sort with both arguments of the same sort,
fmod ASSOCIATIVE-EX1 is
sorts s1 s2 .
subsort s1 < s2 .
op f : s2 s2 -> s2 [assoc] .
op a : -> s1 .
eq f(a,a) = a .
endfm
Maude> red f(a, a) .
reduce in ASSOCIATIVE-EX1 : f(a, a) .
rewrites: 1 in 0ms cpu (0ms real) (~ rews/sec)
result s1: a
or adding enough declarations to cover all possible combinations of arguments; in this case only
one more declaration is enough, as follows:
fmod ASSOCIATIVE-EX2 is
sorts s1 s2 .
subsort s1 < s2 .
op f : s2 s2 -> s2 [assoc] .
op f : s1 s2 -> s2 [assoc] .
op a : -> s1 .
eq f(a,a) = a .
endfm
Maude> red f(a, a) .
reduce in ASSOCIATIVE-EX2 : f(a, a) .
rewrites: 1 in 0ms cpu (0ms real) (~ rews/sec)
result s1: a
When an associative operator is also declared to be commutative using the comm attribute,
Maude computes the commutative completion (switching the order of the argument sorts) of
the given operator declarations before checking preregularity.
In the case of the id: and idem attributes, preregularity modulo those axioms requires that
all collapses, that is, all passages from a term f(t,e) to an equivalent term tby application of an
identity equation f(x,e) = x; or from a term f(t,t) to an equivalent term tby application of an
idempotency equation f(x,x) = x(where in both cases the top function symbol f disappears),
should be such that the least sort of the resulting term tis smaller than or equal to the least sort
of f(t,e) (resp. f(t,t)). A term that can collapse from one sort to a greater or incomparable
sort breaks the sort calculations and violates preregularity modulo such axioms. Therefore,
syntactic conditions ensuring that a collapse is also into a lesser or equal sort are checked by
Maude for both the id: and idem attributes.
21.3.6 Collapse theories
Using id: or idem attributes means that you are (conceptually) working with infinite equiv-
alence classes, and that many lefthand side patterns will match in unexpected ways. Unlike
OBJ3, Maude has true collapse matching algorithms, rather than identity completion, and it
does not try to omit problematic matches. Consider for example the module

<!-- manual page 563 -->
21.3. TRAPS AND KNOWN PROBLEMS 551
fmod COLLAPSE-ID-EX is
sort Foo .
ops a e : -> Foo .
op f : Foo Foo -> Foo [left id: e] .
var X : Foo .
eq f(X, a) = ...
endfm
Then we have
a = f(e, a) = f(e, f(e, a)) = ...
In particular, the pattern f(X, a) matches a with X ←e, leading to possible nontermina-
tion. You should be wary of having an operator with an identity element as the top symbol
for a lefthand side. One useful trick when you need a pattern like f(X, a) is to use a pattern
f(Y, a) where Y has a sort lower than that of the identity element. For example,
fmod COLLAPSE-NAT-EX is
sorts Nat NzNat .
subsort NzNat < Nat .
op 0 : -> Nat .
op s : Nat -> NzNat .
op + : Nat Nat -> Nat [assoc comm id: 0] .
op + : Nat NzNat -> Nat [assoc comm id: 0] .
var X : Nat .
var Y : NzNat .
eq +(s(X), Y) = s(+(X, Y)) .
endfm
Here +(s(X), Y) cannot match s(0) because, although s(0) = +(s(0), 0) by the identity
attribute, Y cannot match 0.
Rewriting with the idem attribute is even riskier. For example,
fmod COLLAPSE-IDEM-EX is
sort Foo .
ops a b : -> Foo .
op f : Foo Foo -> Foo [idem] .
var X : Foo .
eq a = b .
endfm
We then have
a = f(a, a) = f(f(a, a), f(a, a)) = ...
Thus, if acanberewrittenbyanequation, thenanynumberofrewritescanbedonebyusing
the idem axiom to create new copies of a. In fact, the current implementation would choose
the obvious rewrite and just produce b, but this should not be relied upon; COLLAPSE-IDEM-EX
is a nonterminating system. The only safe way to use idem is as follows. Whenever a connected
component is the domain and range of an operator having the idem attribute, then its sorts
are poisoned. Terms of poisoned sorts must never rewrite other than by rules under the control
of a strategy, that is, using metalevel descent functions. Such terms must be built out of free
constructors—operators that may have equational attributes such as comm, but may not have
equations with these operators at the top. Of course, it is ok to have defined functions that
work on such constructor terms; it is just that the terms themselves may not rewrite.

<!-- manual page 564 -->
552 CHAPTER 21. DEBUGGING AND TROUBLESHOOTING
21.3.7 One-sided identities and associativity
When the associativity axiom is combined with a one-sided identity axiom some unexpected
matching properties result. Consider the module:
fmod ASSOC-ID-EX is
sort Foo .
ops a b 1f : -> Foo .
op f : Foo Foo -> Foo [assoc left id: 1f] .
var X Y : Foo .
endfm
Then (see Appendix A.3 for matching commands),
match f(X, Y) <=? f(a, b) .
yields three solutions:
Solution 1
X:Foo --> 1f
Y:Foo --> f(a, b)
Solution 2
X:Foo --> a
Y:Foo --> b
Solution 3
X:Foo --> f(a, 1f)
Y:Foo --> b
whereas the naive user may not have expected the last solution.
Matching with extension can be even more surprising. The command
xmatch f(X, Y) <=? f(a, b) .
yields five solutions:
Solution 1
Matched portion = f(a, 1f)
X:Foo --> a
Y:Foo --> 1f
Solution 2
Matched portion = f(a, 1f)
X:Foo --> f(a, 1f)
Y:Foo --> 1f
Solution 3
Matched portion = (whole)
X:Foo --> 1f
Y:Foo --> f(a, b)
Solution 4
Matched portion = (whole)
X:Foo --> a
Y:Foo --> b

<!-- manual page 565 -->
21.3. TRAPS AND KNOWN PROBLEMS 553
Solution 5
Matched portion = (whole)
X:Foo --> f(a, 1f)
Y:Foo --> b
Here the first two solutions match a portion f(a, 1f) of the subject that was not apparent
from the original problem. However, if one considers the equivalence class of f(a, b) they are
validsolutionsthatarenecessaryforcorrectsimulationof(conditional)rewritingonequivalence
classes.
21.3.8 Memberships for associative operators
Membership axioms can interact with assoc or iter operator attributes in undesirable ways.
The reason is that, for completeness, the operator declarations would have to be tried on
every subterm of every member of the equivalence class, and this is not done (for efficiency
reasons) in the current implementation, giving rise to some warnings.
For associative operators declared at the sort level, membership axioms will be applied
only at the top, they will not be applied to subterms in the process of applying an operator
declaration to compute the sort. For example in the following module
fmod ASSOC-MB-EX1 is
sort Foo .
op f : Foo Foo -> Foo [assoc comm] .
op e : -> [Foo] .
ops a b c d : -> Foo .
mb f(a, e) : Foo .
endfm
the membership axiom will not be used to lower the sort of f(a, f(b, e)) to foo as it does
not match at the top.
Recall from Sections 3.9.3 and 4.8 that terms built with associative operators can be written
in flattened form. This is the notation used for f-terms in the following examples.
Maude> red f(a, b, e) .
Warning: membership axioms are not guaranteed to work correctly for
associative symbol f as it has declarations that are not at the
kind level.
reduce in ASSOC-MB-EX1 : f(e, a, b) .
rewrites: 0 in 0ms cpu (0ms real) (~ rews/sec)
result [Foo]: f(e, a, b)
Maude> red f(a, b, e, a) .
reduce in ASSOC-MB-EX1 : f(e, a, a, b) .
rewrites: 0 in 0ms cpu (0ms real) (~ rews/sec)
result [Foo]: f(e, a, a, b)
Maude> red f(e, b, e, a) .
reduce in ASSOC-MB-EX1 : f(e, e, a, b) .
rewrites: 0 in 0ms cpu (0ms real) (~ rews/sec)
result [Foo]: f(e, e, a, b)
Maude> red f(a, b, e, e, a) .
reduce in ASSOC-MB-EX1 : f(e, e, a, a, b) .

<!-- manual page 566 -->
554 CHAPTER 21. DEBUGGING AND TROUBLESHOOTING
rewrites: 0 in 0ms cpu (0ms real) (~ rews/sec)
result [Foo]: f(e, e, a, a, b)
Maude> red f(a, a, b, e, e, a) .
reduce in ASSOC-MB-EX1 : f(e, e, a, a, a, b) .
rewrites: 0 in 0ms cpu (0ms real) (~ rews/sec)
result [Foo]: f(e, e, a, a, a, b)
Here the intuition is that each e forces the result to the kind level, unless there is an a
to bring it back down. Unfortunately, for f(a, b, e) we would need to use the membership
axiom on a proper subterm, and then use the declaration at the top to arrive at the sort Foo,
and this is not allowed.
Note that the warning produced by Maude is a per module warning and is only printed
once, when the first reduction or rewriting command is given in the module.
The module ASSOC-MB-EX1 above can be rewritten so that sort computations work as ex-
pected as follows:
fmod ASSOC-MB-EX2 is
sort Foo .
op f : [Foo] [Foo] -> [Foo] [assoc comm] .
op e : -> [Foo] .
ops a b c d : -> Foo .
mb f(X:Foo, Y:Foo) : Foo .
mb f(a, e) : Foo .
endfm
Maude> red f(a, b, e) .
reduce in ASSOC-MB-EX2 : f(e, a, b) .
rewrites: 2 in 0ms cpu (1ms real) (~ rews/sec)
result Foo: f(e, a, b)
Maude> red f(a, b, e, a) .
reduce in ASSOC-MB-EX2 : f(e, a, a, b) .
rewrites: 3 in 0ms cpu (0ms real) (~ rews/sec)
result Foo: f(e, a, a, b)
Maude> red f(e, b, e, a) .
reduce in ASSOC-MB-EX2 : f(e, e, a, b) .
rewrites: 6 in 0ms cpu (0ms real) (~ rews/sec)
result [Foo]: f(e, e, a, b)
Maude> red f(a, b, e, e, a) .
reduce in ASSOC-MB-EX2 : f(e, e, a, a, b) .
rewrites: 11 in 0ms cpu (0ms real) (~ rews/sec)
result Foo: f(e, e, a, a, b)
Maude> red f(a, a, b, e, e, a) .
reduce in ASSOC-MB-EX2 : f(e, e, a, a, a, b) .
rewrites: 12 in 0ms cpu (0ms real) (~ rews/sec)
result Foo: f(e, e, a, a, a, b)

<!-- manual page 567 -->
21.3. TRAPS AND KNOWN PROBLEMS 555
Here the operator declaration is at the kind level, and the effect of the declaration of f in
ASSOC-MB-EX1 is obtained by an extra membership axiom.4
Let us see another example of this situation, starting with a module specifying non-empty
lists of natural numbers.
fmod SIMPLE-NAT-LIST is
protecting NAT .
sort NatList .
subsort Nat < NatList .
op __ : NatList NatList -> NatList [assoc] .
endfm
It seems natural to specify sorted lists of natural numbers by importing SIMPLE-NAT-LIST
and then defining a subsort of NatList.
fmod NAIVE-SORTED-NAT-LIST is
protecting SIMPLE-NAT-LIST .
sort SortedNatList .
subsort Nat < SortedNatList < NatList .
vars I J : Nat .
var SNL : SortedNatList .
cmb I J : SortedNatList if I <= J .
cmb I J SNL : SortedNatList if I <= J /\ J SNL : SortedNatList .
endfm
Maude> red 0 1 2 3 4 5 6 7 8 9 .
Warning: membership axioms are not guaranteed to work correctly for
associative symbol __ as it has declarations that are not at the
kind level.
reduce in NAIVE-SORTED-NAT-LIST : 0 1 2 3 4 5 6 7 8 9 .
rewrites: 1354 in 0ms cpu (0ms real) (~ rews/sec)
result SortedNatList: 0 1 2 3 4 5 6 7 8 9
To avoid this, we can rewrite the module above so that we only use kind-level operator
declarations (notice the form of the arrow) and convert all sort-level operator declarations into
memberships.
fmod NAT-LIST-KIND is
protecting NAT .
sort NatList .
subsort Nat < NatList .
op __ : NatList NatList ~> NatList [assoc] .
mb I:NatList J:NatList : NatList .
endfm
fmod SORTED-NAT-LIST-KIND is
protecting NAT-LIST-KIND .
sort SortedNatList .
subsort Nat < SortedNatList < NatList .
4Maude 1 did not allow multiple membership axioms on associative operators. In Maude 3 this works,
althoughitwillbeextremelyinefficientforlargeterms, sincematchingtheextramembershipessentiallyamounts
to expanding out the equivalence class.

<!-- manual page 568 -->
556 CHAPTER 21. DEBUGGING AND TROUBLESHOOTING
vars I J : Nat .
var SNL : SortedNatList .
cmb I J : SortedNatList if I <= J .
cmb I J SNL : SortedNatList if I <= J /\ J SNL : SortedNatList .
endfm
Maude> red 0 1 2 3 4 5 6 7 8 9 .
reduce in SORTED-NAT-LIST-KIND : 0 1 2 3 4 5 6 7 8 9 .
rewrites: 1354 in 0ms cpu (0ms real) (~ rews/sec)
result SortedNatList: 0 1 2 3 4 5 6 7 8 9
21.3.9 Memberships for iterated operators
In analogy to interaction of associative operators and membership declarations, terms con-
structed with a stack of iterated operators may not be assigned the expected sort when it is
necessary to apply a membership axiom to a subterm in order to infer the sort. Again, if
an iter operator is declared at the sort level, Maude will not apply membership axioms to
subterms in order to calculate the sort of a subterm before attempting to apply the operator
declaration to calculate the sort of the whole term. As an example, consider the following
module:
fmod ITER-MB-EX1 is
sort Foo .
op f : Foo -> Foo [iter] .
op e : -> [Foo] .
mb f(e) : Foo .
endfm
Maude> red f(e) .
Warning: membership axioms are not guaranteed to work correctly for
iterated symbol f as it has declarations that are not at the
kind level.
reduce in ITER-MB-EX1 : f(e) .
rewrites: 1 in 0ms cpu (0ms real) (~ rews/sec)
result Foo: f(e)
Maude> red f(f(e)) .
reduce in ITER-MB-EX1 : f^2(e) .
rewrites: 0 in 0ms cpu (0ms real) (~ rews/sec)
result [Foo]: f^2(e)
Maude> red f(f(f(e))) .
reduce in ITER-MB-EX1 : f^3(e) .
rewrites: 0 in 0ms cpu (0ms real) (~ rews/sec)
result [Foo]: f^3(e)
Here the intuition is that e is at the kind level, but f(e) is not. Unfortunately, for f(f(e))
we would need to use the membership axiom on a proper subterm and then use the declaration
at the top to arrive at the sort Foo, and declarations applying above membership axioms for
iterated operators are not allowed.
Again, recall that the warning that membership axioms may not work is only given once
per module. Here it just happens that it is given in response to a reduction command that
does give the right answer.

<!-- manual page 569 -->
21.3. TRAPS AND KNOWN PROBLEMS 557
The example can be rewritten so that membership axioms can be used to compute the
desired sort as follows:
fmod ITER-MB-EX2 is
sort Foo .
op f : [Foo] -> [Foo] [iter] .
op e : -> [Foo] .
mb f(X:Foo) : Foo .
mb f(e) : Foo .
endfm
Maude> red f(e) .
reduce in ITER-MB-EX2 : f(e) .
rewrites: 1 in 0ms cpu (0ms real) (~ rews/sec)
result Foo: f(e)
Maude> red f(f(e)) .
reduce in ITER-MB-EX2 : f^2(e) .
rewrites: 2 in 0ms cpu (0ms real) (~ rews/sec)
result Foo: f^2(e)
Maude> red f(f(f(e))) .
reduce in ITER-MB-EX2 : f^3(e) .
rewrites: 3 in 0ms cpu (0ms real) (~ rews/sec)
result Foo: f^3(e)
Here the operator declaration is at the kind level, and as in the associativity example in the
previous section, the effect of the old declaration is obtained by an extra membership axiom.
Note that using membership axioms in this way loses the efficiency for big towers of operators,
which is the whole point of the iter theory.
21.3.10 Ambiguity in print attribute items
Since Maude has few restrictions on variable names, it is possible to introduce ambiguity with
the print attribute by using strings or attribute names as variables. Consider, for example,
the following module, where the string "here" and the keyword metadata are also declared as
variables.
fmod PRINT-ATTR-AMBIGUOUS is
sort Foo .
op a : -> Foo .
ops f g h : Foo -> Foo .
vars metadata "here" : Foo .
eq f("here") = g("here") [print "here"] .
eq g(metadata) = h(metadata)
[print "metadata = " metadata "g->h equation"] .
endfm
In the print attribute of the first equation, Maude cannot decide whether "here" is a string
constant or a variable. Similarly, in the print attribute of the second equation, Maude will
not be able to decide whether metadata is a variable or a keyword. Under these circumstances,
Maude will output warnings about the multiple parses and then make an undefined choice
between them.

<!-- manual page 570 -->
558 CHAPTER 21. DEBUGGING AND TROUBLESHOOTING
Warning: <standard input>, line 6 (fmod PRINT-ATTR-AMBIGUOUS):
multiple distinct parses for statement
eq f ("here") = g ("here") [print "here"] .
Warning: <standard input>, line 7 (fmod PRINT-ATTR-AMBIGUOUS):
multiple distinct parses for statement
eq g (metadata) = h (metadata)
[print "metadata = " metadata "g->h equation"] .
In this particular example, the equations work but, as a consequence of the ambiguity, the user
does not get the expected information provided by the print attribute.
Maude> set print attribute on .
Maude> red f(g(a)) .
reduce in PRINT-ATTR-AMBIGUOUS : f(g(a)) .
metadata =
h(a)
metadata =
rewrites: 3 in 0ms cpu (0ms real) (25210 rewrites/second)
result Foo: h(h(a))

