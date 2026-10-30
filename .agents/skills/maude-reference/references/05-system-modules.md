# Chapter 5: System Modules (manual pages 97-112)

<!-- manual page 97 -->
Chapter 5
System Modules
A Maude system module specifies a rewrite theory. A rewrite theory has sorts, kinds, and
operators (perhaps with frozen arguments), and can have three types of statements: equations,
memberships, and rules, all of which can be conditional. Therefore, any rewrite theory has
an underlying equational theory, containing the equations and memberships, plus the rules.
What is the intuitive meaning of such rules? Computationally, they specify local concurrent
transitions that can take place in a system if the pattern in the rule’s lefthand side matches a
fragment of the system state and the rule’s condition is satisfied. In that case, the transition
specified by the rule can take place, and the matched fragment of the state is transformed into
the corresponding instance of the righthand side. Logically, that is, when we use rewriting logic
as a logical framework to represent other logics as explained in Section 1.4, a rule specifies a
logical inference rule, and rewriting steps therefore represent inference steps.
As was mentioned in Section 3.2, a system module is declared in Maude using the keywords
mod ⟨ModuleName⟩ is ⟨DeclarationsAndStatements⟩ endm
As for functional modules the first bit of information in the specification is the module’s name,
which must be an identifier. For example,
mod VENDING-MACHINE is
...
endm
where the dots stand for all the declarations and statements in the module, which can be:
1. module importations,
2. sort and subsort declarations,
3. operator declarations,
4. variable declarations,
5. equation and membership statements, and
6. rule statements.
Since declarations (1)–(4) and equational statements (5) are exactly as for functional mod-
ules, all we have left to explain is how rules (conditional or not) are declared. As for equation
and membership statements, rules can be declared with any of the attributes label, metadata,
nonexec, and print (see Section 4.5). However, the owise attribute can only be used with
equations.
85

<!-- manual page 98 -->
86 CHAPTER 5. SYSTEM MODULES
5.1 Unconditional rules
Mathematically, an unconditional rewrite rule has the form l : t →t′, where t,t′ are terms of
the same kind, which may contain variables, and l is the label of the rule. Intuitively, a rule
describes a local concurrent transition in a system: anywhere in the distributed state where a
substitution instance σ(t) of the lefthand side tis found, a local transition of that state fragment
to the new local state σ(t′) can take place. And if many instances of the same or of several
rules can be matched in different nonoverlapping parts of the distributed state, then all of them
can fire concurrently.
An unconditional rule is introduced in Maude with the following syntax:
rl [⟨Label⟩] : ⟨Term-1⟩ => ⟨Term-2⟩ [⟨StatementAttributes⟩] .
As explained in Section 4.5.1, a label can alternatively be declared as a statement attribute;
also, Maude allows declaration of unlabeled rules. In these two cases, the part “[⟨Label⟩] :” is
omitted.
As a first example of a system module we consider the following specification of a vending
machine which dispenses apples and cakes. The module VENDING-MACHINE-SIGNATURE is the
underlyingfunctionalmodule. ThismoduleisimportedbythesystemmoduleVENDING-MACHINE,
which then adds the rules for operating the machine. Although not necessary, in addition to
making the underlying functional module explicit, such splitting of modules can be useful in or-
ganizing a large specification, where a functional part may be shared by several system modules;
see Chapter 7 for a discussion on module importation.
The constants $ and q represent coins of one dollar and one quarter, respectively, while the
constants a and c represent apples and cakes, respectively.
fmod VENDING-MACHINE-SIGNATURE is
sorts Coin Item Marking .
subsorts Coin Item < Marking .
op __ : Marking Marking -> Marking [assoc comm id: null] .
op null : -> Marking .
op $ : -> Coin [format (r! o)] .
op q : -> Coin [format (r! o)] .
op a : -> Item [format (b! o)] .
op c : -> Item [format (b! o)] .
endfm
The format declaration for each constant (see Section 4.4.5) is used to print the constants
using different colors, so that coins can easily be separated from items in a given marking.
mod VENDING-MACHINE is
including VENDING-MACHINE-SIGNATURE .
var M : Marking .
rl [add-q] : M => M q .
rl [add-$] : M => M $ .
rl [buy-c] : $ => c .
rl [buy-a] : $ => a q .
rl [change] : q q q q => $ .
endm
This module specifies a concurrent machine to buy cakes and apples with dollars and quar-
ters. A cake costs a dollar, and an apple three quarters. We can insert dollars and quarters
in the machine, although due to an unfortunate design, the machine only accepts buying cakes
and apples with dollars. When the user buys an apple the machine takes a dollar and returns a
quarter. To alleviate in part this problem, the machine can change four quarters into a dollar.

<!-- manual page 99 -->
5.2. CONDITIONAL RULES 87
The machine is concurrent, because we can push several buttons at once (that is, we can
apply several rules at once), provided enough resources exist in the corresponding slots, called
places. For example, if we have one dollar in the $ place and four quarters in the q place,
we can simultaneously push the buy-a and change buttons, and the machine returns, also
simultaneously, one dollar in $, one apple in a, and one quarter in q.
Note that, since the Maude interpreter is sequential, the above concurrent transitions in the
VENDING-MACHINE module are simulated by corresponding interleavings of sequential rewriting
steps. In a socket-based concurrent implementation, it is possible to execute concurrently many
rewriting steps for a wide range of system modules.1
We might have tried a simpler alternative, namely, using the rule null => q instead of
the add-q rule. However, this would not work. Instead, we have to write M => M q with M
a variable of sort Marking. The reason is that the constant null is not a
__-subterm of any
marking except itself, and therefore it would be impossible to apply the rule null => q with
extension (see Section 4.8).
5.2 Conditional rules
Conditional rewrite rules can have very general conditions involving equations, memberships,
and other rewrites; that is, in their mathematical notation they can be of the form
l: t→t′ if (
ui = vi) ∧(
wj : sj) ∧(
pk →qk)
i
j
k
with no restriction on which new variables may appear in the righthand side or the condition.
There is no need for the condition listing first equations, then memberships, and then rewrites:
this is just a notational abbreviation, since they can be listed in any order. However, in Maude,
conditions are evaluated from left to right, and therefore the order in which they appear,
although mathematically inessential, is very important operationally (see Section 5.3).
In their Maude representation, conditional rules are declared with syntax
crl [⟨Label⟩] : ⟨Term-1⟩ => ⟨Term-2⟩
if ⟨Condition-1⟩ /\ ... /\ ⟨Condition-k⟩
[⟨StatementAttributes⟩] .
where the rule’s label can instead be declared as a statement attribute, or can be omitted
altogether. In either of these two alternatives, the square brackets enclosing the label and the
colon are then omitted.
As in conditional equations, the condition can consist of a single statement or can be a
conjunction formed with the associative connective /\. But now conditions are more general,
since in addition to equations and memberships they can also contain rewrite expressions, for
which the concrete syntax t => t’is used. Furthermore, equations, memberships, and rewrites
can be intermixed in any order. As for functional modules, some of the equations in conditions
can be either matching equations or abbreviated Boolean equations.
We can illustrate the usefulness of rewrite expressions in rule conditions by presenting a
small fragment of a Maude operational semantics for Milner’s CCS language given in [114]:
sorts Label Act Process ActProcess .
subsorts Qid < Label < Act .
subsort Process < ActProcess .
1See [24, Chapter 16] for an interesting example of this kind: a concurrent implementation of a mobile
language entirely programmed in Maude using sockets as external objects in the way explained in Section 9.4.

<!-- manual page 100 -->
88 CHAPTER 5. SYSTEM MODULES
op ~_ : Label -> Label .
op tau : -> Act .
op {_}_ : Act ActProcess -> ActProcess [frozen] .
op _|_ : Process Process -> Process [frozen assoc comm] .
vars P P’ Q Q’ : Process .
var L : Label .
crl [par] : P | Q => {tau} (P’ | Q’)
if P => {L} P’ /\ Q => {~ L} Q’ .
The conditional rule par expresses the synchronized transition of two processes composed
in parallel. The condition of the rule states that the synchronized transition can take place
if one process can perform an action named L and the other can perform the complementary
action named ~ L. In this representation of CCS, the action performed is remembered by the
resulting expression, which is a term of sort ActProcess.
Note the use of the frozen attribute in some of the operators (see Section 4.4.9).
5.3 Admissible system modules
The same way that equations or memberships expressed in their fullest possible generality can-
not be executed by the Maude engine except in a controlled way at the metalevel, conditional
rewrite rules in their fullest generality cannot be executed either, except with a strategy ex-
pressed in the builtin strategy language or at the metalevel. Nonexecutable rules should be
identified by giving them the nonexec attribute.
As for functional modules, the question now becomes: what are the executability require-
ments on the executable statements (i.e., those without the nonexec attribute) of a system
module? It turns out that a quite general class of system modules, called admissible mod-
ules, are executable by Maude’s default interpreter using the rewrite, frewrite, and search
commands, that will be introduced and illustrated in Section 5.4 and are further explained in
Sections A.2 and A.4.
The admissibility requirements for the module’s equations and memberships are exactly as
for functional modules; they were explained in Section 4.6 and are further discussed below.
Two more requirements are needed:
• each executable conditional rule should be admissible, and
• the rules should be coherent relative to the equations, as has already been mentioned in
the introduction.
We explain each of these requirements below.
Given a system module M, a conditional2 rule of the form
l: t→t′ if C1 ∧...∧Cn
such that it does not have the nonexec attribute is called admissible if it satisfies the exact
analogues of the admissibility requirements 1–3 in Section 4.6 for conditional equations, plus
the additional requirement
2For the purposes of this discussion, we view unconditional rules as a special case of conditional rules. The
general admissibility requirement specializes then to a very easy requirement for an unconditional rule t→t′
,
namely, that each variable of t′must appear in t.

<!-- manual page 101 -->
5.3. ADMISSIBLE SYSTEM MODULES 89
4. If Ci is a rewrite ui →u′
i, then
vars(ui) ⊆vars(t) ∪
i−1
j=1
vars(Cj),
and furthermore u′
i is an E(M)-pattern (for the notion of pattern see Section 4.3) for
E(M) the equational theory underlying the module M.
Operationally, we try to satisfy such a rewrite condition by reducing the substitution in-
stance σ(ui) to its canonical form vi with respect to the equations, and then trying to find a
rewrite proof vi →wi (perhaps after many steps of rewriting) with wi in canonical form with
respect to the equations and such that wi is a substitution instance of u′
i. Search for such a wi
is performed by the Maude engine using a breadth-first strategy.
As for functional modules, when executing an admissible conditional rule in a system mod-
ule, the satisfaction of all its conditions is attempted sequentially from left to right; but notice
that now, besides the fact that many matches for the equational conditions may be possible
due to the presence of equational attributes, we also have to deal with the fact that solving
rewrite conditions requires search, including searching for new solutions when previous ones fail
to satisfy subsequent conditions.
We now explain the coherence requirement. A rewrite theory has both rules and equations,
so that rewriting is performed modulo such equations. However, this does not mean that the
Maude implementation must have a matching algorithm for each equational theory that a user
might specify, which is impossible, since matching modulo an arbitrary equational theory is
undecidable.
The equations and memberships specified in a system module M are divided into a set A
of axioms corresponding to equational attributes such as associativity, commutativity, idempo-
tency, and (left-, right- or two-sided) identity declared for different operators in the module (see
Section 4.4.1), for which matching algorithms exist in the Maude implementation, and a set E
of equations and memberships specified in the ordinary way. As already mentioned, for M to
be executable, the set of executable statements in E must be Church-Rosser and terminating
modulo A, or at least ground Church-Rosser and terminating modulo A; that is, we require
that the equational part must be equivalent to an executable functional module.
Moreover, we require that the rules R in the module are coherent [116] with respect to the
equations E modulo A, or at least ground coherent. Coherence means that, given a term t, for
each one-step rewrite of it with some rule in Rmodulo the axioms Ato some term t′, which we
denote t−→1
R/A t′, if u is the canonical term we obtain by rewriting t with the equations and
memberships in E to canonical form modulo A, denoted t −→!
E/A u, then there is a one-step
rewrite of u with some rule in R modulo A, u−→1
R/A u′, such that t′
=E∪A u′, which by the
Church-Rosser and termination properties of E modulo Ais equivalent to t′and u′having the
same canonical form modulo A by E. This requirement is described graphically in Figure 5.1.
Ground coherence is a weaker requirement: we require the exact same diagram to exist only
for ground terms, and E only needs to be ground Church-Rosser and terminating modulo A.
As explained in [116] (for the free case and for coherence modulo associativity and com-
mutativity), for unconditional rules R, coherence can be checked by checking “critical pairs”
between rules Rand equations E, and showing that the corresponding instance of the coherence
diagram can be filled in for all such pairs. That is, we have to look for appropriate overlaps
between lefthand sides of rules and equations using an A-unification algorithm, generate the
corresponding critical pairs, and check their coherence. In the case of ground coherence, it is
not necessary that the critical pairs can be filled in: it is enough to show that each ground
instance of such pairs can be filled in. See Section 7.8 of [24] for an example of a system

<!-- manual page 102 -->
90 CHAPTER 5. SYSTEM MODULES
t 1
R/A//
t′
!
E/A "" w
!
E/A<<
E/A
!
u
1
R/A// u′
Figure 5.1: Coherence diagram
module that is not coherent, a discussion of the critical pairs involved, and a method to make
the specification coherent. See also Section 13.4 of [24] for an example of how coherence can be
checked by critical pair analysis. Similarly, for ground coherence and how to check it, see the
example in Section 11.4.
Why is coherence so important? What does it mean intuitively? Rewriting modulo an
equational theory E∪A, which is what a rewrite theory R= (Σ,E∪A,ϕ,R) really does, is
in general undecidable. The undecidability has to do with the fact that we may need to search
an entire E∪A-equivalence class before we can know if a class representative can be rewritten
with R, that is, if the E∪A-equivalence class can be rewritten. Coherence makes the problem
decidable: all we need to do is to reduce the term to its canonical form by E modulo A, and
then rewrite with R such a canonical form. In a sense, coherence reduces rewriting with R
modulo E∪Ato rewriting with E and Rmodulo A, which is decidable, because we assume we
have an A-matching algorithm.
Could we miss any rewrites that way? Coherence guarantees to us that we could not, since
any rewrite of a term twith Rmust also be possible with t’s canonical form. Maude implicitly
assumes this coherence property. For example, the rewrite command will at each step first
reduce the term to canonical form with E modulo A, and then perform a rewrite step with
R in a rule-fair manner. The frewrite command uses a somewhat different rewrite strategy
to ensure both local fairness and rule fairness, but assumes the same coherence (or ground
coherence) property (see Appendix A.2 and examples in Section 5.4 below).
A last point about the execution of system modules regards frozen argument positions in
operators (see Section 4.4.9). This poses a general constraint on any rewriting strategy what-
soever, including those directly supported by Maude for the rewrite and frewrite commands
(see Section 5.4). The general constraint is that rewriting will never happen below one of the
frozen argument positions in an operator. That is, even though many rewritings may be possi-
ble and there can be a large amount of nondeterminism (so that different rewriting strategies
may lead to quite different results) rewriting under frozen arguments is always forbidden. In
fact, this does not only belong to the module’s operational semantics, but also to the latest
initial model semantics for rewrite theories developed in [16]; we give a brief informal summary
of this semantics below.
Mathematically, a system module, when “flattened” with its imported submodules, exactly
specifies a (generalized) rewrite theory in the sense of [16], that is, a four-tuple
R= (Σ,E∪A,ϕ,R),
where (Σ,E∪A) is the membership equational theory specified by the signature, equational
attributes, and equation and membership statements in the module (just as in the case of
functional modules); ϕis a function, assigning to each operator in Σ the set of natural numbers

<!-- manual page 103 -->
5.4. THE REWRITE, FREWRITE, AND SEARCH COMMANDS 91
corresponding to its frozen arguments (the empty set when no argument is frozen); and Ris the
collection of (possibly conditional) rewrite rules specified in the module and its submodules.
Intuitively, such a rewrite theory specifies a concurrent system. The equational theory
(Σ,E∪A) specifies the “statics” of the system, that is, the algebraic structure of the set3 of
states, whichisspecifiedbytheinitialalgebraTΣ/E∪A. TherulesRandthefreezinginformation
ϕ specify the concurrent system’s “dynamics,” that is, the possible concurrent transitions that
the system can perform. In rewriting logic, such, possibly complex, concurrent transitions
exactly correspond to rewrite proofs; but since several rewrite proofs can indeed correspond to
the same concurrent computation (describing, for example, different semantically equivalent
interleavings), rewriting logic has an equational theory of proof equivalence [91, 16].
The initial model TRof the rewrite theory Rassociates to each kind k a labeled transition
system (in fact, a category) whose set of states is TΣ/E∪A,k, and whose labeled transitions
have the form [α] : [t] →[t′], with [t],[t′] ∈TΣ/E∪A,k, and with [α] an equivalence class of
rewrite proofs modulo the equational theory of proof equivalence. Indeed what the different [α]
represent are the different “truly concurrent” computations of the system specified by R.
5.4 The rewrite, frewrite, and search commands
Now we illustrate the use of the Maude commands available for system modules. Recall the
vending machine example:
mod VENDING-MACHINE is
including VENDING-MACHINE-SIGNATURE .
var M : Marking .
rl [add-q] : M => M q .
rl [add-$] : M => M $ .
rl [buy-c] : $ => c .
rl [buy-a] : $ => a q .
rl [change]: q q q q => $ .
endm
In addition to the show commands discussed in Section 4.9, there is an additional show rls
command for system modules to show the rules of a module. For example, showing the sorts
and the rules of the VENDING-MACHINE module we get:
Maude> show sorts VENDING-MACHINE .
sort Bool .
sort Coin . subsort Coin < Marking .
sort Item . subsort Item < Marking .
sort Marking . subsorts Item Coin < Marking .
Maude> show rls VENDING-MACHINE .
rl [add-q] : M => q M .
rl [add-$] : M => $ M .
rl [buy-c] : $ => c .
rl [buy-a] : $ => q a .
rl [change] : q q q q => $ .
3Moreprecisely, eachkindkinΣ correspondstoadifferentchoiceforasetofstates, namelythesetTΣ/E∪A,k.

<!-- manual page 104 -->
92 CHAPTER 5. SYSTEM MODULES
5.4.1 The rewrite command
We can use the rewrite command (abbreviated rew) to explore the behavior of different initial
markings. The bracketed number between the command and the term to be rewritten provides
an upper bound for the number of rule applications that are allowed.
Maude> rew [1] in VENDING-MACHINE : $ $ q q .
rewrite [1] in VENDING-MACHINE : $ $ q q .
rewrites: 1 in 0ms cpu (9ms real) (~ rews/sec)
result Marking: $ $ q q q
Maude> rew [2] $ $ q q .
rewrite [2] in VENDING-MACHINE : $ $ q q .
rewrites: 2 in 0ms cpu (0ms real) (~ rews/sec)
result Marking: $ $ $ q q q
Maude> rew [3] $ $ q q .
rewrite [3] in VENDING-MACHINE : $ $ q q .
rewrites: 3 in 0ms cpu (0ms real) (~ rews/sec)
result Marking: $ $ $ q q q q
Maude> rew [4] $ $ q q .
rewrite [4] in VENDING-MACHINE : $ $ q q .
rewrites: 4 in 0ms cpu (0ms real) (~ rews/sec)
result Marking: $ $ $ $ q q q q
Maude> rew [5] $ $ q q .
rewrite [5] in VENDING-MACHINE : $ $ q q .
rewrites: 5 in 0ms cpu (0ms real) (~ rews/sec)
result Marking: $ $ $ $ $
Maude> rew [6] $ $ q q .
rewrite [6] in VENDING-MACHINE : $ $ q q .
rewrites: 6 in 0ms cpu (0ms real) (~ rews/sec)
result Marking: $ $ $ $ $ q
Maude> rew [200] $ $ q q .
rewrite [200] in VENDING-MACHINE : $ $ q q .
rewrites: 200 in 10ms cpu (10ms real) (20000 rews/sec)
result Marking: $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $
$ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $
$ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $
$ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ $ q q q
Executing one rewrite starting with two dollars and two quarters, Maude chooses to apply
the add-q rule. If we allow two rewrites Maude applies add-q and then add-$. The third rule
to be applied is add-q again; then, add-$. It goes on applying add-q and add-$ until the rule
change can be applied. The top-down rule-fair rewrite strategy keeps trying to apply rules
on the top operator (__ in this case) in a fair way. The rules applicable at the top are add-q,
add-$, and change, which are tried in this order. Since the top operator is always the same
one, no other rules are used. We can modify the rules buy-c and buy-a so that the lefthand
side has an explicit top level__
as follows:
mod VENDING-MACHINE-TOP is
including VENDING-MACHINE-SIGNATURE .

<!-- manual page 105 -->
5.4. THE REWRITE, FREWRITE, AND SEARCH COMMANDS 93
var M : Marking .
rl [add-q] : M => M q .
rl [add-$] : M => M $ .
rl [buy-c] : $ M => c M .
rl [buy-a] : $ M => a q M .
rl [change]: q q q q => $ .
endm
Now starting with two dollars and two quarters, and executing increasing numbers of rewrites
we see that Maude applies the rules add-$, add-q, buy-c, buy-a, and change.
Maude> rew [2] in VENDING-MACHINE-TOP : $ $ q q .
Advisory: "v.maude", line 18 (mod VENDING-MACHINE-TOP): collapse at
top of $ M may cause it to match more than you expect.
Advisory: "v.maude", line 19 (mod VENDING-MACHINE-TOP): collapse at
top of $ M may cause it to match more than you expect.
rewrite [2] in VENDING-MACHINE-TOP : $ $ q q .
rewrites: 2 in 0ms cpu (0ms real) (~ rews/sec)
result Marking: $ $ $ q q q
Maude> rew [3] $ $ q q .
rewrite [3] in VENDING-MACHINE-TOP : $ $ q q .
rewrites: 3 in 0ms cpu (0ms real) (~ rews/sec)
result Marking: $ $ q q q c
Maude> rew [4] $ $ q q .
rewrite [4] in VENDING-MACHINE-TOP : $ $ q q .
rewrites: 4 in 0ms cpu (0ms real) (~ rews/sec)
result Marking: $ q q q q a c
Maude> rew [5] $ $ q q .
rewrite [5] in VENDING-MACHINE-TOP : $ $ q q .
rewrites: 5 in 0ms cpu (0ms real) (~ rews/sec)
result Marking: $ $ a c
The advisory is about the modified rules for buying. Maude is letting us know that the pattern
$ M will match a term not containing the top-level operator__, when M is instantiated to null.
This is exactly what we want in this case, but it may not always be what the user intended, so
Maude gives you a heads up; see Section 21.3.6 for more details.
Notice that rewriting in VENDING-MACHINE is not terminating. If we remove the rules for
addingcoinsweobtainaterminatingsystemandcanexplorevendingbehaviorusingunbounded
rewriting. Let us consider the following module SIMPLE-VENDING-MACHINE.
mod SIMPLE-VENDING-MACHINE is
including VENDING-MACHINE-SIGNATURE .
rl [buy-c] : $ => c .
rl [buy-a] : $ => a q .
rl [change]: q q q q => $ .
endm
For example, starting with two dollars and rewriting as much as possible we can get an
apple, a cake and a quarter in change.
Maude> rew in SIMPLE-VENDING-MACHINE : $ $ .
rewrite in SIMPLE-VENDING-MACHINE : $ $ .
rewrites: 2 in 0ms cpu (0ms real) (~ rews/sec)

<!-- manual page 106 -->
94 CHAPTER 5. SYSTEM MODULES
result Marking: q a c
Starting with two dollars and three quarters and using only three rewrite rule applications
we get an apple and a cake with a dollar left over.
Maude> rew [3] $ $ q q q .
rewrite [3] in SIMPLE-VENDING-MACHINE : $ $ q q q .
rewrites: 3 in 0ms cpu (0ms real) (~ rews/sec)
result Marking: $ a c
The command continue ⟨Bound⟩(abbreviated cont) tells Maude to continue rewriting
using at most ⟨Bound⟩additional rule applications. For example, we can continue the last
rewrite command (that performed three rewrites) for one more step to get an apple and two
cakes:
Maude> cont 1 .
rewrites: 1 in 0ms cpu (0ms real) (~ rews/sec)
result Marking: a c c
5.4.2 The frewrite command
Let us see now what happens when we use another strategy for rewriting in the original
VENDING-MACHINE module. The frewrite command (abbreviated frew) rewrites a term using
a depth-first position-fair strategy that makes it possible for some rules to be applied that could
be “starved” using the leftmost, outermost rule fair strategy of the rewrite command. The
strategies for the rewrite and frewrite commands are described in detail in Appendix A.2.
Maude> frew [2] in VENDING-MACHINE : $ $ q q .
frewrite [2] in VENDING-MACHINE : $ $ q q .
rewrites: 2 in 0ms cpu (0ms real) (~ rews/sec)
result (sort not calculated): ($ q) ($ $) q q
Maude> frew [12] $ $ q q .
frewrite [12] in VENDING-MACHINE : $ $ q q .
rewrites: 12 in 0ms cpu (0ms real) (~ rews/sec)
result (sort not calculated):
c (q a) ($ q) ($ $) (q q) ($ q) (q q) q q
With two rewrites, one quarter and one dollar are added. With sufficiently many rewrites
(twelve will do), a cake and an apple can be obtained.
In contrast to rewrite, that reduces the whole term using equations after each rule rewrite,
frewrite only reduces the subterm rewritten (to preserve positions not yet visited). Thus,
when rewriting stops, the term may not be fully reduced and hence Maude will not know the
exact least sort of the term yet. This is the reason for the (sort not calculated) comment
in place of a sort in the result line. In the case of a term with an associative and commutative
top operator, the term may not be in its fully flattened form with canonical order of subterms.
This accounts for the parentheses in the result term and the fact that the coins and items are
not listed in order as they are in the result of a rewrite.
The top-down rule-fair strategy of the rewrite command can result in nontermination even
though there is a terminating sequence of rewrites. As an example consider the following
module:
mod BB-TEST is
sort Expression .

<!-- manual page 107 -->
5.4. THE REWRITE, FREWRITE, AND SEARCH COMMANDS 95
ops a b bingo : -> Expression .
op f : Expression Expression -> Expression .
rl a => b .
rl b => a .
rl f(b, b) => bingo .
endm
Giving the rewrite command with input term f(a, a) will result in the following looping
computation:
f(a, a) => f(b, a) => f(a, a) => f(b, a) => f(a, a) => ...
This is because using the top-down rule-fair strategy of the rewrite command, the third rule
always fails to match and never gets a chance to be applied. As already mentioned above, the
frewrite command uses on the other hand a position-fair bottom-up strategy that makes it
possible for other rules to be applied. As a consequence, some rewriting computations that
could be nonterminating using the rewrite command become terminating with frewrite. For
example, using the frewrite command in place of rewrite in the above example we get
Maude> frew in BB-TEST : f(a, a) .
frewrite in BB-TEST : f(a, a) .
rewrites: 3 in 0ms cpu (0ms real) (~ rews/sec)
result Expression: bingo
5.4.3 The search command
The rewrite and frewrite commands each explore just one possible behavior (sequence of
rewrites) of a system described by a set of rewrite rules and an initial state. The search
command allows one to explore (following a breadth-first strategy) the reachable state space in
different ways. Its syntax conforms to the following general scheme
search [ n, m ] in ⟨ModId⟩ : ⟨Term-1⟩ ⟨SearchArrow⟩ ⟨Term-2⟩
such that ⟨Condition⟩.
where
• n is an optional argument providing a bound on the number of desired solutions;
• m is another optional argument stating the maximum depth of the search;
• the module ⟨ModId⟩where the search takes place can be omitted;
• ⟨Term-1⟩is the starting term;
• ⟨Term-2⟩is the pattern that has to be reached;
• ⟨SearchArrow⟩is an arrow indicating the form of the rewriting proof from ⟨Term-1⟩until
⟨Term-2⟩:
– =>1 means a rewriting proof consisting of exactly one step,
– =>+ means a rewriting proof consisting of one or more steps,
– =>* means a proof consisting of none, one, or more steps, and
– =>! indicates that only canonical final states are allowed, that is, states that cannot
be further rewritten, and
– =># searches for states having multiple distinct successors.
The one step arrow =>1 is an abbreviation of the one-or-more steps arrow =>+ with the
depth bound m set to 1.
• ⟨Condition⟩states an optional property that has to be satisfied by the reached state; the
syntactic form of the condition is the same as the one of conditions for conditional rules
(see Section 5.2).

<!-- manual page 108 -->
96 CHAPTER 5. SYSTEM MODULES
For example, for our finite vending machine, SIMPLE-VENDING-MACHINE, we can use the
search command to answer the question: if I have a dollar and three quarters, can I get a cake
and an apple? This is done by searching for states that match a corresponding pattern. In this
example, we use the =>! symbol, meaning that we are searching for terminal states, that is, for
states that cannot be further rewritten. Moreover, no bound in the number of solutions or in
the depth of the search is needed.
Maude> search in SIMPLE-VENDING-MACHINE :
$ q q q =>! a c M:Marking .
search in SIMPLE-VENDING-MACHINE : $ q q q =>! a c M:Marking .
Solution 1 (state 4)
states: 6 rewrites: 5 in 0ms cpu (0ms real) (~ rews/sec)
M:Marking --> null
No more solutions.
states: 6 rewrites: 5 in 0ms cpu (1ms real) (~ rews/sec)
The answer is yes, and the desired state is numbered 4. To see the sequence of rewrites that
allowed us to reach this state we can type
Maude> show path 4 .
state 0, Marking: $ q q q
===[ rl [buy-a] : $ => q a . ]===>
state 2, Marking: q q q q a
===[ rl [change] : q q q q => $ . ]===>
state 3, Marking: $ a
===[ rl [buy-c] : $ => c . ]===>
state 4, Marking: a c
One can get just the sequence of labels of applied rules with a similar command:
Maude> show path labels 4 .
buy-a
change
buy-c
The last version of this command shows the sequence of states and the label of each transition,
but does not show the complete rule statement.
Maude> show path states 4 .
state 0, Marking: $ q q q
--- buy-a --->
state 2, Marking: q q q q a
--- change --->
state 3, Marking: $ a
--- buy-c --->
state 4, Marking: a c
It is also possible to print out the current search graph generated by a search command
using the command show search graph. After the above search we get
Maude> show search graph .
state 0, Marking: $ q q q
arc 0 ===> state 1 (rl [buy-c] : $ => c .)
arc 1 ===> state 2 (rl [buy-a] : $ => q a .)

<!-- manual page 109 -->
5.4. THE REWRITE, FREWRITE, AND SEARCH COMMANDS 97
state 0: $ q q q
buy-c
buy-a
state 1: q q q c
state 2: q q q q a
change
state 3: $ a
buy-c
buy-a
state 4: a c
state 5: q a a
Figure 5.2: Graphical representation of search graph in example
state 1, Marking: q q q c
state 2, Marking: q q q q a
arc 0 ===> state 3 (rl [change] : q q q q => $ .)
state 3, Marking: $ a
arc 0 ===> state 4 (rl [buy-c] : $ => c .)
arc 1 ===> state 5 (rl [buy-a] : $ => q a .)
state 4, Marking: a c
state 5, Marking: q a a
This search graph is represented graphically in Figure 5.2.
From the same initial state, $ q q q, we can see if it is possible to reach a final state with
an apple and more things, learning that there are exactly two possibilities:
Maude> search $ q q q =>! a M:Marking such that M:Marking =/= null .
search in SIMPLE-VENDING-MACHINE : $ q q q =>! a M:Marking
such that M:Marking =/= null = true .
Solution 1 (state 4)
states: 6 rewrites: 6 in 0ms cpu (0ms real) (~ rews/sec)
M:Marking --> c
Solution 2 (state 5)
states: 6 rewrites: 7 in 0ms cpu (0ms real) (~ rews/sec)
M:Marking --> q a
No more solutions.
states: 6 rewrites: 7 in 0ms cpu (0ms real) (~ rews/sec)
In the following example with a different initial state, namely, $ q q q q, we are looking
for intermediate states from which it is possible to get later either two apples (and two quarters
left) or two cakes, getting exactly one solution.

<!-- manual page 110 -->
98 CHAPTER 5. SYSTEM MODULES
Maude> search $ q q q q =>+ M:Marking
such that M:Marking => a a q q /\ M:Marking => c c .
search in SIMPLE-VENDING-MACHINE : $ q q q q =>+ M:Marking
such that M:Marking => q q a a /\ M:Marking => c c .
Solution 1 (state 1)
states: 2 rewrites: 10 in 0ms cpu (0ms real) (96153 rewrites/second)
M:Marking --> $ $
No more solutions.
states: 9 rewrites: 38 in 0ms cpu (0ms real) (95477 rewrites/second)
Sometimes it is necessary to impose a limit on the number of solutions searched for, since
in general the number of such solutions could be infinite. In the previous examples there were
only one or two solutions, so imposing a bound would not make any difference. But, returning
to the coin generating (and thus nonterminating) vending machine module VENDING-MACHINE,
the search space becomes now infinite, so it is important to be able to limit either the number
of solutions sought or the depth of the search, or both.
We can look for different ways to use a dollar and three quarters to buy an apple and two
cakes. First we ask for one solution, and then use the bounded continue command to see
another solution. Note that here we use the search mode =>+, which means searching for states
reachable by at least one rewrite. Searching for terminal states in the VENDING-MACHINEmodule
is futile!
Maude> search [1] in VENDING-MACHINE : $ q q q =>+ a c c M:Marking .
search in VENDING-MACHINE : $ q q q =>+ a c c M .
Solution 1 (state 108)
states: 109 rewrites: 1857 in 0ms cpu (41ms real)(~rews/sec)
M --> q q q q
Maude> cont 1 .
Solution 2 (state 113)
states: 114 rewrites: 185 in 0ms cpu (4ms real) (~ rews/sec)
M --> null
We can also use the maximum depth optional argument, but if we put a too small depth,
we do not get any solution:
Maude> search [, 4] $ q q q =>+ a c c M:Marking .
search [, 4] in VENDING-MACHINE : $ q q q =>+ a c c M .
No solution.
states: 66 rewrites: 875 in 10ms cpu (3ms real) (87500 rews/sec)
By increasing the depth to 10 we will get 98 solutions. If we are interested in only a few of
those, we can set both bounds, like in the following example:
Maude> search [4, 10] $ q q q =>+ a c c M:Marking .
search [4, 10] in VENDING-MACHINE : $ q q q =>+ a c c M .
Solution 1 (state 108)
states: 109 rewrites: 1857 in 0ms cpu (7ms real) (~ rews/sec)
M --> q q q q
Solution 2 (state 113)

<!-- manual page 111 -->
5.4. THE REWRITE, FREWRITE, AND SEARCH COMMANDS 99
states: 114 rewrites: 2042 in 0ms cpu (7ms real) (~ rews/sec)
M --> null
Solution 3 (state 160)
states: 161 rewrites: 3328 in 0ms cpu (11ms real) (~ rews/sec)
M --> q q q q q
Solution 4 (state 164)
states: 165 rewrites: 3524 in 0ms cpu (12ms real) (~ rews/sec)
M --> q
IfweinsistnowinthemarkingMbeingdifferentfromnull, thenoneoftheprevioussolutions
is discarded, but we still get four solutions:
Maude> search [4, 10] $ q q q =>+ a c c M:Marking
such that M:Marking =/= null .
search [4, 10] in VENDING-MACHINE : $ q q q =>+ a c c M
such that M =/= null = true .
Solution 1 (state 108)
states: 109 rewrites: 1858 in 0ms cpu (5ms real) (~ rews/sec)
M --> q q q q
Solution 2 (state 160)
states: 161 rewrites: 3331 in 10ms cpu (13ms real) (333100 rews/sec)
M --> q q q q q
Solution 3 (state 164)
states: 165 rewrites: 3528 in 10ms cpu (14ms real) (352800 rews/sec)
M --> q
Solution 4 (state 175)
states: 176 rewrites: 3904 in 10ms cpu (15ms real) (390400 rews/sec)
M --> $ q q q q
Let us end up our examples on the search command illustrating the =># option. Consider
the following command:
Maude> search $ q q q =># M:Marking .
search in SIMPLE-VENDING-MACHINE : $ q q q =># M:Marking .
Solution 1 (state 0)
states: 3 rewrites: 2 in 0ms cpu (0ms real) (2000000 rewrites/second)
M:Marking --> $ q q q
Solution 2 (state 3)
states: 6 rewrites: 5 in 0ms cpu (0ms real) (116279 rewrites/second)
M:Marking --> $ a
No more solutions.
states: 6 rewrites: 5 in 0ms cpu (0ms real) (62500 rewrites/second)
The command explores the entire state space reachable from the term $ q q q. As solutions
it gives those states that can be rewritten in more than one way. In this case, every time we
have a $, there are two rules that can be applied, to buy either a cake or an apple.

<!-- manual page 112 -->
100 CHAPTER 5. SYSTEM MODULES
In Chapter 11 we will see how the search command can be used to model check invariant
properties of a concurrent system specified in Maude as a system module.
In case you forget to set a bound on the search command or on its continuation, you can
also interrupt a search in progress by typing control-C. In this case one of two things will
happen, depending on what Maude is doing at the instant you hit control-C. If Maude is not
doing a rewrite, the command will exit. If Maude is doing a rewrite, you will end up in the
debugger. In this latter case it is probably best to type abort, since the debugger has no special
support for search at the moment. See Section 21.1.3 and Appendix A.14 for more information
on the debugger.
The full syntax and different options for the search command and for all the other com-
mands illustrated in this section are explained in detail in Appendix A.

