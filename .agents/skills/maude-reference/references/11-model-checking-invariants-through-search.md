# Chapter 11: Model Checking Invariants Through Search (manual pages 331-340)

<!-- manual page 331 -->
Chapter 11
Model Checking Invariants
Through Search
A rewrite theory, specified in Maude as a system module, provides an executable mathematical
model of a concurrent system. We can use the Maude specification to simulate the concurrent
system so specified. But we can do more. Under appropriate conditions we can check that
our mathematical model satisfies some important properties, or obtain a useful counterexample
showing that the property in question is violated. This kind of model-checking analysis can be
quite general. Chapter 12 will explain how, under appropriate finite reachability assumptions,
we can model check any linear time temporal logic (LTL) property of a system specified in
Maude as a system module. This chapter focuses on a simpler, yet very useful, model-checking
capability, namely, the model checking of invariants, which can be accomplished just by using
the search command.
11.1 Invariants
Invariants are the most common and useful safety properties, that is, properties stating that
something bad should never happen. Given a transition system and an initial state s0, an
invariant I is a predicate defining a subset of states meeting two properties:
• it contains s0, and
• it contains any state reachable from s0 through a finite number of transitions.
Therefore, aninvariantisapredicatedefiningasetofstatesthatcontainsallthestatesreachable
from s0. If an invariant holds, that is, if it is truly an invariant satisfying the two properties
above, then we know that something “bad” can never happen, namely, the negation¬I of the
invariant is impossible. In other words, we view¬I as a bad property that should never happen,
and I as a good property we want to ensure.
Given a rewrite theory R= (Σ,E,ϕ,R) specified in Maude as a system module, we can
define an invariant I, yielding a decidable set of states, by:
1. choosing a given kind k in Σ as the kind of states and an initial state init in it; and
2. specifying an equationally-defined Boolean condition I, so that the invariant holds of a
state if and only if such a state satisfies the condition I.
The transition system implicit in this setting is one in which a one-step transition between
two states, that is, between two elements [t],[t′] ∈ TΣ/E,k, exists if and only if there is a
representative t0 ∈[t] and a one-step rewrite with the rules R, t0 −→1 t′
0, such that t′
0 ∈[t′].
319

<!-- manual page 332 -->
320 CHAPTER 11. MODEL CHECKING INVARIANTS THROUGH SEARCH
We introduce the notation
R,init |= PI
to state that the transition system associated with Rwith state kind k and initial state init
satisfies the invariant I.
11.2 Model checking of invariants
Thekeyquestionnowis: howcanweautomaticallyverify thataninvariantIholds? Theanswer
is very simple. Assuming that R= (Σ,E,ϕ,R) satisfies reasonable executability conditions,
namely, that E and R are finite sets, (Σ,E) is ground Church-Rosser and terminating,1 and
the rules Rare ground coherent with E, and assuming, further, that all the conditions for rules
in R are purely equational, then I holds if and only if the search command
search init =>* x:k such that I(x:k) =/= true .
has no solutions. Indeed, having no solutions exactly means that on init, and on all states
reachable from it, the predicate I evaluates to true, that is, that I is an invariant. Assuming
that I has been fully defined in all cases (which is always easy with the owise feature, described
in Section 4.5.4), so that it always evaluates to either true or false, we could instead give the
command
search init =>* x:k such that not I(x:k) .
Consider, for example, a simple clock that marks the hours of the day. Such a clock can be
specified with the system module
mod SIMPLE-CLOCK is
protecting INT .
sort Clock .
op clock : Int -> Clock [ctor] .
var T : Int .
rl clock(T) => clock((T + 1) rem 24) .
endm
A natural initial state is the state clock(0). Note that, in principle, the clock could be in an
infinite number of states, such as clock(7633157) or clock(-33457129). The point, however,
is that if our initial state is clock(0), then only states clock(T) with times T such that
0 <= T < 24 can be reached. This suggests making the predicate 0 <= T < 24 an invariant of
our clock system.
Using simple linear arithmetic reasoning, we can express the negation of such an invariant
as the predicate (T < 0) or (T >= 24); thus, we can automatically verify that our simple
clock satisfies the invariant by giving the command:
Maude> search in SIMPLE-CLOCK : clock(0) =>* clock(T)
such that T < 0 or T >= 24 .
No solution.
states: 24 rewrites: 216 in 0ms cpu (2ms real) (~ rews/sec)
If, as it is the case in this clock example, the number of states reachable from the initial state
is finite, then search commands of this kind provide a decision procedure for the satisfaction of
invariants. That is, in finite time Maude will either find no solutions to a search for a state
1As usual, the ground Church-Rosser and termination assumptions should be understood modulo any axioms
A⊆E which in Maude are declared as equational attributes of operators.

<!-- manual page 333 -->
11.2. MODEL CHECKING OF INVARIANTS 321
violating the invariant, or will find a state violating the invariant together with a sequence of
rewrites from the initial state to it, that is, a counterexample.
But what if the number of states reachable from the initial state is infinite? In such a
case, if the invariant I is violated, the search command will terminate in finite time yielding
a counterexample. Termination is guaranteed by the breadth-first nature of the search. The
point is that such a counterexample is a reachable state; therefore, there is a finite sequence of
rewrites from the initial state to such a violating state. Since there is only a finite number of
rules R, and therefore a finite number of ways that each state can be rewritten, even though the
number of reachable states is infinite, the number of states reachable from the initial state by
a sequence of rewrites of length less than a given bound is always finite. This bounded subset
is always explored in finite time by the search command. This means that, for systems where
the set of reachable states is infinite, search becomes a semi-decision procedure for detecting
the violation of an invariant. That is, if the invariant is violated, we are guaranteed to get a
counterexample; but, if it is not violated, we will search forever, never finding it.
Wecanillustratethesemi-decisionprocedurenatureofsearchfortheverificationofinvariant
failures with a simple infinite-state example of processes and resources. This example has some
similarities with the classical dining philosophers problem, but it is on the one hand simpler
(processes and resources have no identities or topology), and on the other hand more complex,
since the number of processes and resources can grow dynamically in an unbounded manner.
mod PROCS-RESOURCES is
sorts State Resources Process .
subsorts Process Resources < State .
ops res null : -> Resources [ctor] .
op p : Resources -> Process [ctor] .
op __ : Resources Resources -> Resources
[ctor assoc comm id: null] .
op __ : State State -> State [ctor ditto] .
rl [acq1] : p(null) res => p(res) .
rl [acq2] : p(res) res => p(res res) .
rl [rel] : p(res res) => p(null) res res .
rl [dupl] : p(null) res => p(null) res p(null) res .
endm
The state is a soup (multiset) of processes and resources. Each process needs to acquire two
resources. Originally, each process p contains the null state; but if a resource res is available,
it can acquire it (rule acq1). If a second resource becomes available, it can also acquire it (rule
acq2). After acquiring both resources and using them, the process can release them (rule rel).
Furthermore, the number of processes and resources can grow in an unbounded manner by the
duplication of each process-resource pair (rule dupl).
One invariant we might like to verify about this system is deadlock freedom. There are two
ways to model check this property: one completely straightforward, and another requiring some
extra work. The straightforward manner is to give the search command
Maude> search in PROCS-RESOURCES : res p(null) =>! X:State .
Solution 1 (state 1)
states: 3 rewrites: 2 in 0ms cpu (0ms real) (~ rews/sec)
X:State --> p(res)
Solution 2 (state 5)
states: 9 rewrites: 9 in 0ms cpu (1ms real) (~ rews/sec)

<!-- manual page 334 -->
322 CHAPTER 11. MODEL CHECKING INVARIANTS THROUGH SEARCH
X:State --> p(res) p(res)
Solution 3 (state 13)
states: 19 rewrites: 26 in 0ms cpu (3ms real) (~ rews/sec)
X:State --> p(res) p(res) p(res)
Solution 4 (state 25)
states: 34 rewrites: 56 in 0ms cpu (4ms real) (~ rews/sec)
X:State --> p(res) p(res) p(res) p(res)
Solution 5 (state 43)
states: 55 rewrites: 104 in 0ms cpu (23ms real) (~ rews/sec)
X:State --> p(res) p(res) p(res) p(res) p(res)
......
Solution 20 (state 1649)
states: 1770 rewrites: 5640 in 20ms cpu (67ms real)
(282000 rews/sec)
X:State --> p(res) p(res) p(res) p(res) p(res) p(res) p(res) p(res)
p(res) p(res) p(res) p(res) p(res) p(res) p(res) p(res)
p(res) p(res) p(res) p(res)
......
Maude will indeed continue printing all the solutions it finds. But since there is an infinite
number of deadlock states, it may be preferable to specify in advance a bound on the number of
solutions, giving, for example, a command like the following, that looks for at most 5 solutions.
Maude> search [5] in PROCS-RESOURCES : res p(null) =>! X:State .
The nice thing about model checking deadlock freedom this way is that there is no need
to explicitly specify the invariant as a Boolean predicate. This is because the negation of the
invariant is by definition the set of deadlock states, which is what the search command with
the =>! qualification precisely looks for.
If one wishes, one can, with a little more work, perform an equivalent model checking of
the same property by using an explicit enabledness predicate. Of course, this cannot be done
in the original module, because such a predicate has not been defined, but this is easy enough
to do:
mod PROCS-RESOURCES-ENABLED is
protecting PROCS-RESOURCES .
protecting BOOL .
op enabled : State -> Bool .
eq enabled(p(null) res X:State) = true .
eq enabled(p(res) res X:State) = true .
eq enabled(p(res res) X:State) = true .
eq enabled(X:State) = false [owise] .
endm
One can then give the command
Maude> search [5] in PROCS-RESOURCES-ENABLED : res p(null)
=>* X:State such that enabled(X:State) =/= true .
getting the following 5 solutions:

<!-- manual page 335 -->
11.3. BOUNDED MODEL CHECKING OF INVARIANTS 323
Solution 1 (state 1)
states: 2 rewrites: 4 in 0ms cpu (0ms real) (~ rews/sec)
X:State --> p(res)
Solution 2 (state 5)
states: 6 rewrites: 15 in 0ms cpu (0ms real) (~ rews/sec)
X:State --> p(res) p(res)
Solution 3 (state 13)
states: 14 rewrites: 41 in 0ms cpu (0ms real) (~ rews/sec)
X:State --> p(res) p(res) p(res)
Solution 4 (state 25)
states: 26 rewrites: 87 in 0ms cpu (1ms real) (~ rews/sec)
X:State --> p(res) p(res) p(res) p(res)
Solution 5 (state 43)
states: 44 rewrites: 160 in 0ms cpu (1ms real) (~ rews/sec)
X:State --> p(res) p(res) p(res) p(res) p(res)
11.3 Bounded model checking of invariants
In cases where either the set of reachable states is infinite, or it is finite but too large to be
explored in reasonable time due to time and memory limitations, bounded model checking is an
appealing formal analysis method. The idea of bounded model checking is that we model check
a property, not for all reachable states, but only for those states reachable within a certain
depth bound, that is, reachable by a sequence of transitions of bounded length. Of course, our
analysis is not complete anymore, since we may fail to find a counterexample lying at greater
depth. However, bounded model checking can be quite effective in finding counterexamples and
it is a widely used procedure. Bounded model checking of invariants is supported in Maude by
means of the bounded search command.
Consider the following specification of a readers-writers system.
mod READERS-WRITERS is
sort Nat .
op 0 : -> Nat [ctor] .
op s : Nat -> Nat [ctor] .
sort Config .
op <_,_> : Nat Nat -> Config [ctor] . --- readers/writers
vars R W : Nat .
rl < 0, 0 > => < 0, s(0) > .
rl < R, s(W) > => < R, W > .
rl < R, 0 > => < s(R), 0 > .
rl < s(R), W > => < R, W > .
endm
A state is represented by a tuple < R, W > indicating the number R of readers and the number
W of writers accessing a critical resource. Readers and writers can leave the resource at any
time, but writers can only gain access to it if nobody else is using it, and readers only if there
are no writers.

<!-- manual page 336 -->
324 CHAPTER 11. MODEL CHECKING INVARIANTS THROUGH SEARCH
Taking < 0, 0 > as the initial state, this simple system satisfies two important invariants,
namely:
• mutual exclusion: readers and writers never access the resource simultaneously: only
readers or only writers can do so at any given time.
• one writer: at most one writer will be able to access the resource at any given time.
We could try model checking these two invariants in two different ways:
• wecanrepresenttheinvariantsimplicitly byrepresentingtheirnegationsthroughpatterns;
or
• we can represent them explicitly as Boolean predicates.
To model check our two invariants using an implicit representation we could use the commands
Maude> search < 0, 0 > =>* < s(N:Nat), s(M:Nat) > .
Maude> search < 0, 0 > =>* < N:Nat, s(s(M:Nat)) > .
sincethenegationofthefirstinvariantcorrespondstothesimultaneouspresenceofbothreaders
and writers, which is exactly captured by the first pattern < s(N:Nat), s(M:Nat) >; whereas
the negation of the fact that zero or at most one writer should be present at any given time is
exactly captured by the second pattern < N:Nat, s(s(M:Nat)) >.
Theproblemwiththeabovetwosearchcomandsisthat, sincethenumberorreadersallowed
is unbounded, the set of reachable states is infinite and the commands never terminate. We
can instead perform bounded model checking of these two invariants by giving a depth bound,
for example 105, with the commands:
Maude> search [1, 100000] in READERS-WRITERS : < 0, 0 > =>* < s(N:Nat), s(M:Nat) > .
No solution.
states: 100002 rewrites: 200001 in 312460ms cpu (636669ms real) (640 rews/sec)
Maude> search [1, 100000] in READERS-WRITERS : < 0, 0 > =>* < N:Nat, s(s(M:Nat)) > .
No solution.
states: 100002 rewrites: 200001 in 70600ms cpu (623434ms real) (2832 rews/sec)
As the reader can observe, these computations take quite a long time. Notice that the terms
appearing during the search grow very quickly. A very simple way of improving performance
in this case is using the iter attribute for the s operator.
op s : Nat -> Nat [ctor iter] .
Then, we obtain a much better performance:
Maude> search [1, 100000] in READERS-WRITERS : < 0, 0 > =>* < s(N:Nat), s(M:Nat) > .
No solution.
states: 100002 rewrites: 200001 in 610ms cpu (1298ms real) (327870 rews/sec)
Maude> search [1, 100000] in READERS-WRITERS : < 0, 0 > =>* < N:Nat, s(s(M:Nat)) > .
No solution.
states: 100002 rewrites: 200001 in 400ms cpu (1191ms real) (500002 rews/sec)
In the following section we will use some formal tools for checking properties about the
READERS-WRITERS module. Since some of these tools cannot handle the iter attribute, we use
the module as shown above.

<!-- manual page 337 -->
11.4. VERIFYING INFINITE-STATE SYSTEMS THROUGH ABSTRACTIONS 325
11.4 Verifying infinite-state systems through abstractions
The bounded model checking of our two invariants for the readers and writers example up
to depth 106 greatly increases our confidence that the invariants hold, but, as already men-
tioned, bounded model checking is an incomplete procedure that falls short of being a proof: a
counterexample at greater depth could exist.
Can we actually fully verify such invariants in an automatic way? One possible method is
to verify the invariants through search not on the original infinite-state system, but on a finite-
state abstraction of it, that is, on an appropriate quotient of the original system whose set of
reachable states is finite. The paper [99] describes a simple method for, given a rewrite theory
R= (Σ,E,ϕ,R), defining an abstraction Aof it: just add equations. That is, we can define
our abstract theory as a rewrite theory A= (Σ,E∪G,ϕ,R), with G a set of extra equations
powerful enough to collapse the infinite set of reachable states into a finite set. This method
can be used (with an additional deadlock-freedom requirement) to verify not just invariants,
but in fact any LTL formula (see [99] and Section 13.4 of [24]).
Of course, the equations Gwe add cannot be arbitrary. First of all, they should respect all
the necessary executability assumptions, so that in A= (Σ,E∪G,ϕ,R) the equations E∪G
should be ground Church-Rosser and terminating,2 and the rules Rshould be ground coherent
with E∪G. Furthermore, the equations G should be invariant-preserving with respect to the
invariants that we want to model check; that is, for any such invariant I and for any two ground
terms t,t′denoting states such that t=E∪G t′, it should be the case that E ⊢I(t) = I(t′).
A first key observation is that, if both Rand Aprotect the sort Bool, that is, the only
canonical forms of that sort are true and false, and true ̸= false, then the equations G are
invariant-preserving. A second key observation is that we may be able to automatically check
that a module protects the sort Bool by:
(i) checking that it has no equations with true or false in the lefthand side;
(ii) checking that it is terminating with the Maude Termination Tool (MTT) [47, 48, 38, 39];
(iii) checking that it is ground confluent and sort-decreasing with the Church-Rosser Checker
(CRC) tool [47, 48, 45, 46]; and
(iv) checking that it is sufficiently complete with the Sufficient Completeness Checker (SCC)
tool [47, 48, 72].
Indeed, since true and false are the only constructors of sort Bool, (i)–(iii) ensure the “no
confusion,” true ̸= false part, whereas (iv) ensures the “no junk” part of the protection.
For invariant verification, the key property that an abstraction meeting these requirements
satisfies is that, if I is one of the invariants preserved by G, then we have the implication
A,init |= PI=⇒ R,init |= PI
Therefore, if we can verify the invariant on A, we are sure that it also holds on R. However, the
fact that we find a counterexample in Adoes not necessarily mean that a counterexample exists
for R: it could be a spurious counterexample, caused by Abeing too coarse of an abstraction,
and therefore having no counterpart in R. In such cases, one possible approach is to refine our
abstraction by imposing fewer equations.
We can illustrate these ideas by defining an abstraction of our readers-writers system. In
order to check that the equations in our abstraction preserve the invariants, we need an explicit
representation of those invariants.
mod READERS-WRITERS-PREDS is
protecting READERS-WRITERS .
2Again, possibly modulo equational attributes A⊆E∪G.

<!-- manual page 338 -->
326 CHAPTER 11. MODEL CHECKING INVARIANTS THROUGH SEARCH
ops mutex one-writer : Config -> Bool [frozen] .
eq mutex(< s(N:Nat), s(M:Nat) >) = false .
eq mutex(< 0, N:Nat >) = true .
eq mutex(< N:Nat, 0 >) = true .
eq one-writer(< N:Nat, s(s(M:Nat)) >) = false .
eq one-writer(< N:Nat, 0 >) = true .
eq one-writer(< N:Nat, s(0) >) = true .
endm
We can now define our abstraction, in which all the states having more than one reader and
no writers are identified with the state having exactly one reader and no writer.
mod READERS-WRITERS-ABS is
including READERS-WRITERS-PREDS .
including READERS-WRITERS .
eq < s(s(N:Nat)), 0 > = < s(0), 0 > .
endm
wherethesecondincludingimportationisneededbecausetheREADERS-WRITERSmoduleisnot
protected,butwouldbeassumedprotectedbydefault(becauseitisprotectedinREADERS-WRITERS-PREDS)
unless we explicitly declare it in including mode (see Section 7.1.4).
In order to check both the executability and the invariant-preservation properties of this
abstraction, since we have no equations with either true or false in their lefthand side, we
now just need to check:
1. thattheequationsinbothREADERS-WRITERS-PREDSandREADERS-WRITERS-ABSareground
confluent, sort-decreasing, and terminating;
2. that the equations in both READERS-WRITERS-PREDS and READERS-WRITERS-ABS are suf-
ficiently complete; and
3. that the rules in both READERS-WRITERS-PREDS and READERS-WRITERS-ABS are ground
coherent with respect to their equations.
Regarding termination, since the equations of READERS-WRITERS-ABS contain those of the
module READERS-WRITERS-PREDS, it is enough to check the termination of the equations in the
former. The MTT tool, using the AProVE termination tool as backend, checks this automati-
cally.
We can check the confluence of the equational part of the READERS-WRITERS-ABS and
READERS-WRITERS-PREDS modules using the CRC as follows:
Maude> (check Church-Rosser READERS-WRITERS-PREDS .)
Church-Rosser checking of READERS-WRITERS-PREDS
Checking solution:
All critical pairs have been joined. The specification is locally-confluent.
The specification is sort-decreasing.
Maude> (check Church-Rosser READERS-WRITERS-ABS .)
Church-Rosser checking of READERS-WRITERS-ABS
Checking solution:
All critical pairs have been joined. The specification is locally-confluent.
The specification is sort-decreasing.
which finishes task (1).
Regarding (2), the SCC tool gives us:
Maude> (scc READERS-WRITERS-PREDS .)
Checking sufficient completeness of READERS-WRITERS-PREDS ...
Success: READERS-WRITERS-PREDS is sufficiently complete under the

<!-- manual page 339 -->
11.4. VERIFYING INFINITE-STATE SYSTEMS THROUGH ABSTRACTIONS 327
assumption that it is weakly-normalizing, confluent, and
sort-decreasing.
Maude> (scc READERS-WRITERS-ABS .)
Checking sufficient completeness of READERS-WRITERS-ABS ...
Success: READERS-WRITERS-ABS is sufficiently complete under the
assumption that it is weakly-normalizing, confluent, and
sort-decreasing.
This leaves us with task (3), where the Coherence Checker (ChC) tool responds as follows:
Maude> (check coherence READERS-WRITERS-PREDS .)
Coherence checking of READERS-WRITERS-PREDS
Coherence checking solution:
All critical pairs have been rewritten and all equations
are non-constructor.
The specification is coherent.
Maude> (check coherence READERS-WRITERS-ABS .)
Coherence checking of READERS-WRITERS-ABS
Coherence checking solution:
The following critical pairs cannot be rewritten:
cp < s(0), 0 > => < s(N*@:Nat), 0 > .
Of course, ground coherence means that all ground instances of this pair can be rewritten by a
one-step rewrite up to canonical form by the equations. We can reason by cases and decompose
this critical pair into two:
cp < s(0), 0 > => < s(0), 0 > .
cp < s(0), 0 > => < s(s(N:Nat)), 0 > .
Usingthereducecommandwecancheckthatthecanonicalformoftheterm< s(s(N:Nat)), 0 >
is < s(0), 0 >. Therefore, all we need to do is to check that < s(0), 0 > rewrites to itself
(up to canonical form) in one step. We can do this check by giving the command:
Maude> search in READERS-WRITERS-ABS : < s(0), 0 > =>1 X:Config .
Solution 1 (state 0)
states: 1 rewrites: 2 in 0ms cpu (26ms real) (~ rews/sec)
X:Config --> < s(0), 0 >
Solution 2 (state 1)
states: 2 rewrites: 3 in 0ms cpu (124ms real) (~ rews/sec)
X:Config --> < 0, 0 >
No more solutions.
We have therefore completed all the checks (1)–(3) and can now automatically verify the
two invariants on the abstract system, and therefore conclude that they hold in our original
infinite-state readers-writers system, by executing the search commands:
Maude> search in READERS-WRITERS-ABS :
< 0, 0 > =>* C:Config such that mutex(C:Config) = true .
No solution.
states: 3 rewrites: 9 in 0ms cpu (0ms real) (~ rews/sec)

<!-- manual page 340 -->
328 CHAPTER 11. MODEL CHECKING INVARIANTS THROUGH SEARCH
Maude> search in READERS-WRITERS-ABS :
< 0, 0 > =>* C:Config such that one-writer(C:Config) = false .
No solution.
states: 3 rewrites: 9 in 0ms cpu (0ms real) (~ rews/sec)

