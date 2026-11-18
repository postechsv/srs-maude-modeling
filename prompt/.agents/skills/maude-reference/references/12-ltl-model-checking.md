# Chapter 12: LTL Model Checking (manual pages 341-364)

<!-- manual page 341 -->
Chapter 12
LTL Model Checking
As pointed out in Section 1.4, given a Maude system module, we can distinguish two levels of
specification:
• a system specification level, provided by the rewrite theory specified by that system mod-
ule which defines the behavior of the system, and
• a property specification level, given by some property (or properties) φ that we want to
state and prove about our module.
This chapter discusses how a specific property specification logic, linear temporal logic
(LTL), and a decision procedure for it, model checking, can be used to prove properties when
the set of states reachable from an initial state in a system module is finite. It also explains how
this is supported in Maude by its MODEL-CHECKER module, and other related modules, including
the SAT-SOLVER module that can be used to check both satisfiability of an LTL formula and
LTL tautologies. These modules are found in the file model-checker.maude. In Section 12.6
we consider a simple, but useful, extension of LTL that we call LTL+ and whose model checking
is also supported by Maude.
Temporal logic allows specification of properties such as safety properties (ensuring that
somethingbadneverhappens)andlivenessproperties(ensuringthatsomethinggoodeventually
happens). These properties are related to the infinite behavior of a system. There are different
temporal logics [19]; we focus on linear temporal logic [85, 19], because of its intuitive appeal,
widespread use, and well-developed proof methods and decision procedures.
12.1 LTL formulas and the LTL module
Given a set AP of atomic propositions, we define the formulas of the propositional linear tem-
poral logic LTL(AP) inductively as follows:
• True: ⊤∈LTL(AP).
• Atomic propositions: If p∈AP, then p∈LTL(AP).
• Next operator: If φ∈LTL(AP), then ⃝φ∈LTL(AP).
• Until operator: If φ,ψ∈LTL(AP), then φUψ∈LTL(AP).
• Boolean connectives: If φ,ψ ∈LTL(AP), then the formulas¬φ, and φ∨ψ are in
LTL(AP).
Other LTL connectives can be defined in terms of the above minimal set of connectives as
follows:
• Other Boolean connectives:
– False: ⊥= ¬⊤
329

<!-- manual page 342 -->
330 CHAPTER 12. LTL MODEL CHECKING
– Conjunction: φ∧ψ= ¬((¬φ) ∨(¬ψ))
– Implication: φ→ψ= (¬φ) ∨ψ.
• Other temporal operators:
– Eventually: Qφ= ⊤Uφ
– Henceforth: Pφ= ¬Q¬φ
– Release: φRψ= ¬((¬φ) U(¬ψ))
– Unless: φWψ= (φUψ) ∨(Pφ)
– Leads-to: φY ψ= P(φ→(Qψ))
– Strong implication: φ⇒ψ= P(φ→ψ)
– Strong equivalence: φ⇔ψ= P(φ↔ψ).
The LTL syntax, in a typewriter approximation of the above mathematical syntax, is sup-
ported in Maude by the following LTL functional module (in the file model-checker.maude).
fmod LTL is
protecting BOOL .
sorts Formula .
*** primitive LTL operators
ops True False : -> Formula [ctor format (g o)] .
op ~_ : Formula -> Formula [ctor prec 53 format (r o d)] .
op _/\_ : Formula Formula -> Formula
[comm ctor gather (E e) prec 55 format (d r o d)] .
op _\/_ : Formula Formula -> Formula
[comm ctor gather (E e) prec 59 format (d r o d)] .
op O_ : Formula -> Formula [ctor prec 53 format (r o d)] .
op _U_ : Formula Formula -> Formula
[ctor prec 63 format (d r o d)] .
op _R_ : Formula Formula -> Formula
[ctor prec 63 format (d r o d)] .
*** defined LTL operators
op _->_ : Formula Formula -> Formula
[gather (e E) prec 65 format (d r o d)] .
op _<->_ : Formula Formula -> Formula [prec 65 format (d r o d)] .
op <>_ : Formula -> Formula [prec 53 format (r o d)] .
op []_ : Formula -> Formula [prec 53 format (r d o d)] .
op _W_ : Formula Formula -> Formula [prec 63 format (d r o d)] .
op _|->_ : Formula Formula -> Formula [prec 63 format (d r o d)] .
op _=>_ : Formula Formula -> Formula
[gather (e E) prec 65 format (d r o d)] .
op _<=>_ : Formula Formula -> Formula [prec 65 format (d r o d)] .
vars f g : Formula .
eq f -> g = ~ f \/ g .
eq f <-> g = (f -> g) /\ (g -> f) .
eq <> f = True U f .
eq [] f = False R f .
eq f W g = (f U g) \/ [] f .
eq f |-> g = [](f -> (<> g)) .
eq f => g = [] (f -> g) .
eq f <=> g = [] (f <-> g) .

<!-- manual page 343 -->
12.2. ASSOCIATING KRIPKE STRUCTURES TO REWRITE THEORIES 331
*** negative normal form
eq ~ True = False .
eq ~ False = True .
eq ~ ~ f = f .
eq ~ (f \/ g) = ~ f /\ ~ g .
eq ~ (f /\ g) = ~ f \/ ~ g .
eq ~ O f = O ~ f .
eq ~ (f U g) = (~ f) R (~ g) .
eq ~ (f R g) = (~ f) U (~ g) .
endfm
Note that, for the moment, no set AP of atomic propositions has been specified in the LTL
module. Section 12.2 explains how such atomic propositions are defined for a given system
module M, and Section 12.3 explains how they are added to the LTL module as a subsort Prop
of Formula in the MODEL-CHECKER module.
Note that the nonconstructor connectives have been defined in terms of more basic con-
structor connectives in the first set of equations. But since there are good reasons to put an
LTL formula in negative normal form by pushing the negations next to the atomic propositions
(this is specified by the second set of equations) we need to consider also the duals of the basic
connectives ⊤, ⃝, U, and ∨(respectively, True, O_, _U_, and _\/_) as constructors. That is,
we need to also have as constructors the dual connectives: ⊥, R, and ∧(respectively, False,
_R_, and _/\_). Note that ⃝is self-dual.
12.2 Associating Kripke structures to rewrite theories
Since the models of temporal logic are Kripke structures, we need to explain how we can
associate a Kripke structure to the rewrite theory specified by a Maude system module M.
Kripke structures are the natural models for propositional temporal logic. Essentially, a
Kripke structure is a (total) unlabeled transition system to which we have added a collection of
unary state predicates on its set of states.
A binary relation R ⊆A×A on a set A is called total if and only if for each a ∈A there
is at least one a′∈A such that (a,a′) ∈R. If R is not total, it can be made total by defining
R•
= R∪{(a,a) ∈A2 |̸∃a′∈A(a,a′) ∈R}.
A Kripke structure is a triple A= (A,→A,L) such that A is a set, called the set of states,
→A is a total binary relation on A, called the transition relation, and L : A −→P(AP) is
a function, called the labeling function, associating to each state a ∈A the set L(a) of those
atomic propositions in AP that hold in the state a.
The semantics of LTL(AP) is defined by means of a satisfaction relation
A,a|= φ
between a Kripke structure Ahaving AP as its atomic propositions, a state a∈A, and an LTL
formula φ∈LTL(AP). Specifically, A,a|= φ holds if and only if for each path π ∈Path(A)a
the path satisfaction relation
A,π|= φ
holds, where we define the set Path(A)a of computation paths starting at state a as the set
of functions of the form π : N −→ A such that π(0) = a and, for each n ∈ N, we have
π(n) →Aπ(n+ 1).
Wecandefinethepathsatisfactionrelation(foranypath,beginningatanystate)inductively
as follows:

<!-- manual page 344 -->
332 CHAPTER 12. LTL MODEL CHECKING
• We always have A,π|=LTL ⊤.
• For p∈AP,
A,π|=LTL p ⇔ p∈L(π(0)).
• For ⃝φ∈LTL(A),
A,π|=LTL ⃝φ ⇔ A,s; π|=LTL φ,
where s: N −→N is the successor function, and where (s; π)(n) = π(s(n)) = π(n+ 1).
• For φUψ∈LTL(A),
A,π|=LTL φUψ ⇔
(∃n∈N)
((A,sn; π|=LTL ψ) ∧((∀m∈N) m<n ⇒A,sm; π|=LTL φ)).
• For¬φ∈LTL(AP),
A,π|=LTL ¬φ ⇔ A,π̸|=LTL φ.
• For φ∨ψ∈LTL(AP),
A,π|=LTL φ∨ψ ⇔ A,π|=LTL φ or A,π|=LTL ψ.
How can we associate a Kripke structure to the rewrite theory R= (Σ,E,ϕ,R) specified
by a Maude system module M? We just need to make explicit two things:
• the intended kind k of states in the signature Σ, and
• the relevant state predicates, that is, the relevant set AP of atomic propositions.
In general, the state predicates need not be part of the system specification and therefore
they need not be specified in our system module M. They are typically part of the property spec-
ification. This is because the state predicates need not be related to the operational semantics
of M: they are just certain predicates about the states of the system specified by M that are
needed to specify some properties.
Therefore, after choosing a given kind, say [Foo], in M as our kind for states we can specify
the relevant state predicates in a module M-PREDS protecting M according to the following
general pattern:
mod M-PREDS is
protecting M .
including SATISFACTION .
subsort Foo < State .
...
endm
where the dots ‘...’ indicate the part in which the syntax and semantics of the relevant state
predicates are specified, as further explained in what follows.
The module SATISFACTION (contained in the model-checker.maude file) is very simple,
and has the following specification:
fmod SATISFACTION is
protecting BOOL .
sorts State Prop .
op _|=_ : State Prop ~> Bool [frozen] .
endfm
where the sorts State and Prop are unspecified. However, by importing SATISFACTION into
M-PREDS and giving the subsort declaration
subsort Foo < State .

<!-- manual page 345 -->
12.2. ASSOCIATING KRIPKE STRUCTURES TO REWRITE THEORIES 333
all terms of sort Foo in M are also made terms of sort State. Note that we then have the kind
identity [Foo]= [State].
The operator
op _|=_ : State Prop ~> Bool [frozen] .
is crucial to define the semantics of the relevant state predicates in M-PREDS. Each such state
predicate is declared as an operator of sort Prop. In standard LTL propositional logic, the set
AP of atomic propositions is assumed to be a set of constants. In Maude, however, we can
define parametric state predicates, that is, operators of sort Prop which need not be constants,
but may have one or more sorts as parameter arguments. We then define the semantics of such
state predicates (when the predicate holds) by appropriate equations.
Note that the operator _|=_ is declared as partial, using ~> (see Section 3.5). As we will
see below, a user is only required to ensure that it reduces to true in the positive case(s) and
it can return anything in the negative cases. Declaring it to be partial means that it returns a
result in the kind [Bool] rather than sort Bool, thus protecting BOOL, as it is declared in the
module SATISFACTION.
We can illustrate all this by means of a simple mutual exclusion example. Suppose that our
original system module M is the following module MUTEX, in which two processes, one named a
and another named b, can be either waiting or in their critical section, and take turns accessing
their critical section by passing each other a different token (either $ or *).
mod MUTEX is
sorts Name Mode Proc Token Conf .
subsorts Token Proc < Conf .
op none : -> Conf [ctor] .
op __ : Conf Conf -> Conf [ctor assoc comm id: none] .
ops a b : -> Name [ctor] .
ops wait critical : -> Mode [ctor] .
op [_,_] : Name Mode -> Proc [ctor] .
ops * $ : -> Token [ctor] .
rl [a-enter] : $ [a, wait] => [a, critical] .
rl [b-enter] : * [b, wait] => [b, critical] .
rl [a-exit] : [a, critical] => [a, wait] * .
rl [b-exit] : [b, critical] => [b, wait] $ .
endm
Our obvious kind for states is the kind [Conf] of configurations. In order to state the
desired safety and liveness properties we need state predicates telling us whether a process is
waiting or is in its critical section. We can make these predicates parametric on the name of
the process and define their semantics as follows:
mod MUTEX-PREDS is
protecting MUTEX .
including SATISFACTION .
subsort Conf < State .
op crit : Name -> Prop .
op wait : Name -> Prop .
var N : Name .
var C : Conf .
var P : Prop .
eq [N, critical] C |= crit(N) = true .
eq [N, wait] C |= wait(N) = true .
eq C |= P = false [owise] .
endm

<!-- manual page 346 -->
334 CHAPTER 12. LTL MODEL CHECKING
Note the equations, defining when each of the two parametric state predicates holds in a given
state.
The above example illustrates a general method by which desired state predicates for a
module M are defined in a protecting extension, say M-PREDS, of M which imports SATISFACTION.
One specifies the desired states by choosing a sort in M and declaring it as a subsort of State.
One then defines the syntax of the desired state predicates as operators of sort Prop, and
defines their semantics by means of a set of equations that specify for what states a given
state predicate evaluates to true. We assume that those equations, when added to those of
M, are (ground) Church-Rosser and terminating, and, furthermore, that M’s equational part is
protected when the new operators and equations defining the state predicates are added.
Since we should protect BOOL, it is important to make sure that satisfaction of state predi-
cates is fully defined. This can be checked with Maude’s Sufficient Completeness Checker (SCC)
tool. This means that we should give equations for when the predicates are true and when
they are false. In practice, however, this can often be reduced to specifying when a predicate
is true by means of (possibly conditional) equations of the general form,
t|= p(v1,...,vn) = true if C
because we can use the owise feature (described in Section 4.5.4) to cover all the remaining
cases, when it is false, with an equation
x: State |= p(y1,...,yn) = false [owise].
In some cases, however—for example, because we want to perform reasoning using formal tools
which do not accept owise declarations—we may fully define the true and false cases of a
predicate not by using the owise attribute, but by explicit (possibly conditional) equations of
the more general form,
t|= p(v1,...,vn) = bexp if C,
where bexp is an arbitrary Boolean expression.
We are now ready to associate with a system module M specifying a rewrite theory R=
(Σ,E,ϕ,R) (with a selected kind k of states and with state predicates Π defined by means of
equations D in a protecting extension M-PREDS) a Kripke structure whose atomic predicates
are specified by the set
APΠ = {θ(p) |p∈Π, θground substitution},
wherebyconventionweusethesimplifiednotationθ(p) todenotethegroundtermθ(p(x1,...,xn)).
This defines a labeling function LΠ on the set of states TΣ/E,k assigning to each [t] ∈TΣ/E,k
the set of atomic propositions
LΠ([t]) = {θ(p) ∈APΠ |(E∪D) ⊢ (∀∅) t|= θ(p) = true}.
The Kripke structure we are interested in is then
K(R,k)Π = (TΣ/E,k,(→1
R)•,LΠ),
where (→1
R)•denotes the total relation extending the one-step R-rewriting relation →1
Ramong
states of kind k, that is, [t] →1
R [t′] holds if and only if there are u ∈[t] and u′ ∈[t′] such
that u′ is the result of applying one of the rules in R to u at some position. Under the usual
assumptions that E is (ground) Church-Rosser and terminating (possibly modulo some axioms
Acontained in E) and R is (ground) coherent relative to E (again, possibly modulo A), ucan
always be chosen to be the canonical form of t under the equations E.

<!-- manual page 347 -->
12.3. LTL MODEL CHECKING 335
12.3 LTL model checking
Suppose that, given a system module M specifying a rewrite theory R= (Σ,E,ϕ,R), we have:
• chosen a kind k in M as our kind of states;
• defined some state predicates Π and their semantics in a module, say M-PREDS, protecting
M by the method already explained in Section 12.2.
Then, as explained in Section 12.2, this defines a Kripke structure K(R,k)Π on the set of atomic
propositions APΠ. Given an initial state [t] ∈TΣ/E,k and an LTL formula φ∈LTL(APΠ) we
would like to have a procedure to decide the satisfaction relation
K(R,k)Π,[t] |= φ.
In general this relation is undecidable. It becomes decidable if two conditions hold:
1. the set of states in TΣ/E,k that are reachable from [t] by rewriting is finite,
1 and
2. the rewrite theory R= (Σ,E,ϕ,R) specified by M plus the equations D defining the
predicates Π are such that:
• both E and E∪D are (ground) Church-Rosser and terminating, perhaps modulo
some axioms A, with (Σ,E) ⊆(Σ ∪Π,E∪D) a protecting extension, and
• R is (ground) coherent relative to E (again, perhaps modulo some axioms A).
Under these assumptions, both the state predicates Π and the transition relation →1
R are
computable and, given the finite reachability assumption, we can then settle the above satisfac-
tion problem using a model-checking procedure.
Specifically, Maude uses an on-the-fly LTL model-checking procedure of the style described
in [19]. The basis of this procedure is the following. Each LTL formula φ has an associated
Büchi automaton Bφ whose acceptance ω-language is exactly that of the behaviors satisfying
φ. We can then reduce the satisfaction problem
K(R,k)Π,[t] |= φ
to the emptiness problem of the language accepted by the synchronous product of B¬φ and
(the Büchi automaton associated with) (K(R,k)Π,[t]). The formula φ is satisfied if and only
if such a language is empty. The model-checking procedure checks emptiness by looking for
a counterexample, that is, an infinite computation belonging to the language recognized by
the synchronous product. For a detailed explanation of this general method of on-the-fly LTL
model checking, see [19]; and for a discussion of the specific techniques used in the Maude LTL
model-checker implementation, see [53].
Thismakesclearourinterestinobtainingthenegative normal form ofaformula¬φ, sincewe
needittobuildtheBüchiautomatonB¬φ. ForefficiencypurposesweneedtomakeB¬φ assmall
as possible. The following module LTL-SIMPLIFIER (also in the model-checker.maude file)
tries to further simplify the negative normal form of the formula¬φ in the hope of generating
a smaller Büchi automaton B¬φ. This module is optional (the user may choose to include it or
not when doing model checking) but tends to help building a smaller B¬φ.
fmod LTL-SIMPLIFIER is
including LTL .
*** The simplifier is based on:
*** *** Kousha Etessami and Gerard J. Holzman,
"Optimizing Buchi Automata", CONCUR 2000, LNCS 1877.
*** We use the Maude sort system to do much of the work.
1Note that this can happen even when TΣ/E,k is an infinite set: there is no requirement that TΣ/E,k is finite.

<!-- manual page 348 -->
336 CHAPTER 12. LTL MODEL CHECKING
sorts TrueFormula FalseFormula PureFormula PE-Formula PU-Formula .
subsort TrueFormula FalseFormula < PureFormula <
PE-Formula PU-Formula < Formula .
op True : -> TrueFormula [ctor ditto] .
op False : -> FalseFormula [ctor ditto] .
op _/\_ : PE-Formula PE-Formula -> PE-Formula [ctor ditto] .
op _/\_ : PU-Formula PU-Formula -> PU-Formula [ctor ditto] .
op _/\_ : PureFormula PureFormula -> PureFormula [ctor ditto] .
op _\/_ : PE-Formula PE-Formula -> PE-Formula [ctor ditto] .
op _\/_ : PU-Formula PU-Formula -> PU-Formula [ctor ditto] .
op _\/_ : PureFormula PureFormula -> PureFormula [ctor ditto] .
op O_ : PE-Formula -> PE-Formula [ctor ditto] .
op O_ : PU-Formula -> PU-Formula [ctor ditto] .
op O_ : PureFormula -> PureFormula [ctor ditto] .
op _U_ : PE-Formula PE-Formula -> PE-Formula [ctor ditto] .
op _U_ : PU-Formula PU-Formula -> PU-Formula [ctor ditto] .
op _U_ : PureFormula PureFormula -> PureFormula [ctor ditto] .
op _U_ : TrueFormula Formula -> PE-Formula [ctor ditto] .
op _U_ : TrueFormula PU-Formula -> PureFormula [ctor ditto] .
op _R_ : PE-Formula PE-Formula -> PE-Formula [ctor ditto] .
op _R_ : PU-Formula PU-Formula -> PU-Formula [ctor ditto] .
op _R_ : PureFormula PureFormula -> PureFormula [ctor ditto] .
op _R_ : FalseFormula Formula -> PU-Formula [ctor ditto] .
op _R_ : FalseFormula PE-Formula -> PureFormula [ctor ditto] .
vars p q r s : Formula .
var pe : PE-Formula .
var pu : PU-Formula .
var pr : PureFormula .
*** Rules 1, 2 and 3; each with its dual.
eq (p U r) /\ (q U r) = (p /\ q) U r .
eq (p R r) \/ (q R r) = (p \/ q) R r .
eq (p U q) \/ (p U r) = p U (q \/ r) .
eq (p R q) /\ (p R r) = p R (q /\ r) .
eq True U (p U q) = True U q .
eq False R (p R q) = False R q .
*** Rules 4 and 5 do most of the work.
eq p U pe = pe .
eq p R pu = pu .
*** An extra rule in the same style.
eq O pr = pr .
*** We also use the rules from:
*** *** *** Fabio Somenzi and Roderick Bloem,
"Efficient Buchi Automata from LTL Formulae",
p247-263, CAV 2000, LNCS 1633.
*** that are not subsumed by the previous system.
*** Four pairs of duals.

<!-- manual page 349 -->
12.3. LTL MODEL CHECKING 337
eq O p /\ O q = O (p /\ q) .
eq O p \/ O q = O (p \/ q) .
eq O p U O q = O (p U q) .
eq O p R O q = O (p R q) .
eq True U O p = O (True U p) .
eq False R O p = O (False R p) .
eq (False R (True U p)) \/ (False R (True U q))
= False R (True U (p \/ q)) .
eq (True U (False R p)) /\ (True U (False R q))
= True U (False R (p /\ q)) .
*** <= relation on formula
op _<=_ : Formula Formula -> Bool [prec 75] .
eq p <= p = true .
eq False <= p = true .
eq p <= True = true .
ceq p <= (q /\ r) = true if (p <= q) /\ (p <= r) .
ceq p <= (q \/ r) = true if p <= q .
ceq (p /\ q) <= r = true if p <= r .
ceq (p \/ q) <= r = true if (p <= r) /\ (q <= r) .
ceq p <= (q U r) = true if p <= r .
ceq (p R q) <= r = true if q <= r .
ceq (p U q) <= r = true if (p <= r) /\ (q <= r) .
ceq p <= (q R r) = true if (p <= q) /\ (p <= r) .
ceq (p U q) <= (r U s) = true if (p <= r) /\ (q <= s) .
ceq (p R q) <= (r R s) = true if (p <= r) /\ (q <= s) .
*** conditional rules depending on <= relation
ceq p /\ q = p if p <= q .
ceq p \/ q = q if p <= q .
ceq p /\ q = False if p <= ~ q .
ceq p \/ q = True if ~ p <= q .
ceq p U q = q if p <= q .
ceq p R q = q if q <= p .
ceq p U q = True U q if p =/= True /\ ~ q <= p .
ceq p R q = False R q if p =/= False /\ q <= ~ p .
ceq p U (q U r) = q U r if p <= q .
ceq p R (q R r) = q R r if q <= p .
endfm
Suppose that all the requirements listed above to perform model checking are satisfied. How
do we then model check a given LTL formula in Maude for a given initial state [t] in a module
M? We define a new module, say M-CHECK, according to the following pattern:
mod M-CHECK is
protecting M-PREDS .
including MODEL-CHECKER .
including LTL-SIMPLIFIER . *** optional
op init : -> k . *** optional
eq init = t . *** optional
endm

<!-- manual page 350 -->
338 CHAPTER 12. LTL MODEL CHECKING
The declaration of a constant init of the kind of states is not necessary: it is a matter of
convenience, since the initial state t may be a large term.
The module MODEL-CHECKER (in the file model-checker.maude) is as follows:
fmod MODEL-CHECKER is
protecting QID .
including SATISFACTION .
including LTL .
subsort Prop < Formula .
*** transitions and results
sorts RuleName Transition TransitionList ModelCheckResult .
subsort Qid < RuleName .
subsort Transition < TransitionList .
subsort Bool < ModelCheckResult .
ops unlabeled deadlock : -> RuleName .
op {_,_} : State RuleName -> Transition [ctor] .
op nil : -> TransitionList [ctor] .
op __ : TransitionList TransitionList -> TransitionList
[ctor assoc id: nil] .
op counterexample :
TransitionList TransitionList -> ModelCheckResult [ctor] .
op modelCheck : State Formula ~> ModelCheckResult
[special (...)] .
endfm
Its key operator is modelCheck (whose special attribute has been omitted here), which
takes a state and an LTL formula and returns either the Boolean true if the formula is sat-
isfied, or a counterexample when it is not satisfied. Note that the sort Prop coming from the
SATISFACTION module is declared as a subsort of Formula in LTL. In each concrete use of
MODEL-CHECKER, such as that in M-CHECK above, the atomic propositions in Prop will have been
specified in a module such as M-PREDS.
Let us illustrate the use of this operator with our MUTEX example. Following the pattern
described above, we can define the module
mod MUTEX-CHECK is
protecting MUTEX-PREDS .
including MODEL-CHECKER .
including LTL-SIMPLIFIER .
ops initial1 initial2 : -> Conf .
eq initial1 = $ [a, wait] [b, wait] .
eq initial2 = * [a, wait] [b, wait] .
endm
The relationships between all the modules involved at this point is illustrated in Figure 12.1,
where a single arrow represents an including importation and a triple arrow represents a
protecting importation.
We are then ready to model check different LTL properties of MUTEX. The first obvious
property to check is mutual exclusion:
Maude> red modelCheck(initial1, [] ~(crit(a) /\ crit(b))) .
reduce in MUTEX-CHECK :
modelCheck(initial1, []~ (crit(a) /\ crit(b))) .
result Bool: true

<!-- manual page 351 -->
12.3. LTL MODEL CHECKING 339
MUTEX-CHECK
LTL-SIMPLIFIER
MODEL-CHECKER
MUTEX-PREDS
LTL QID
SATISFACTION
MUTEX
BOOL
Figure 12.1: Importation graph of model-checking modules
Maude> red modelCheck(initial2, [] ~(crit(a) /\ crit(b))) .
reduce in MUTEX-CHECK :
modelCheck(initial2, []~ (crit(a) /\ crit(b))) .
result Bool: true
We can also model check the strong liveness property that if a process waits infinitely often,
then it is in its critical section infinitely often:
Maude> red modelCheck(initial1, ([]<> wait(a)) -> ([]<> crit(a))) .
reduce in MUTEX-CHECK :
modelCheck(initial1, []<> wait(a) -> []<> crit(a)) .
result Bool: true
Maude> red modelCheck(initial1, ([]<> wait(b)) -> ([]<> crit(b))) .
reduce in MUTEX-CHECK :
modelCheck(initial1, []<> wait(b) -> []<> crit(b)) .
result Bool: true
Maude> red modelCheck(initial2, ([]<> wait(a)) -> ([]<> crit(a))) .
reduce in MUTEX-CHECK :
modelCheck(initial2, []<> wait(a) -> []<> crit(a)) .
result Bool: true
Maude> red modelCheck(initial2, ([]<> wait(b)) -> ([]<> crit(b))) .
reduce in MUTEX-CHECK :
modelCheck(initial2, []<> wait(b) -> []<> crit(b)) .
result Bool: true
Of course, not all properties are true. Therefore, instead of a success we can get a coun-
terexample showing why a property fails. Suppose that we want to check whether, beginning
in the state initial1, process b will always be waiting. We then get the counterexample:

<!-- manual page 352 -->
340 CHAPTER 12. LTL MODEL CHECKING
Maude> red modelCheck(initial1, [] wait(b)) .
reduce in MUTEX-CHECK : modelCheck(initial1, []wait(b)) .
result ModelCheckResult:
counterexample({$ [a, wait] [b, wait], ’a-enter}
{[a, critical] [b, wait], ’a-exit}
{* [a, wait] [b, wait], ’b-enter},
{[a, wait] [b, critical], ’b-exit}
{$ [a, wait] [b, wait], ’a-enter}
{[a, critical] [b, wait], ’a-exit}
{* [a, wait] [b, wait], ’b-enter})
The main constructors used in the syntax of a counterexample term are:
op {_,_} : State RuleName -> Transition .
op nil : -> TransitionList [ctor] .
op __ : TransitionList TransitionList -> TransitionList
[ctor assoc id: nil] .
op counterexample :
TransitionList TransitionList -> ModelCheckResult [ctor] .
Therefore, a counterexample is a pair consisting of two lists of transitions, where the first
corresponds to a finite path beginning in the initial state, and the second describes a loop. This
is because, if an LTL formula φis not satisfied by a finite Kripke structure, it is always possible
to find a counterexample for φhaving the form of a path of transitions followed by a cycle [19].
Note that each transition is represented as a pair, consisting of a state and the label of the rule
applied to reach the next state.
Let us finish this section with an example of how not to use the model checker. Consider
the following specification:
mod MODEL-CHECK-BAD-EX is
including MODEL-CHECKER .
extending LTL .
sort Foo .
op a : -> Foo .
op f : Foo -> Foo .
rl a => f(a) .
subsort Foo < State .
op p : -> Prop .
endm
This module describes an infinite transition system of the form
a →f(a) →f(f(a)) →f(f(f(a))) →···,
where the property p is not satisfied anywhere. Therefore the state a does not satisfy, e.g., the
property []p. However, if we try to invoke Maude with the command
Maude> red in MODEL-CHECK-BAD-EX : modelCheck(a, []p) .
we run into a nonterminating computation.
Making explicit that p does not hold in a by adding the equation
eq (a |= p) = false .
does not help. We run into the same problem even if the formula does not contain a temporal
operator: modelCheck(a, p) does not terminate either. The reason is that the set of states
reachable from a is not finite, and hence one of the main requirements for the model-checking
algorithm is not satisfied.

<!-- manual page 353 -->
12.4. THE LTL SATISFIABILITY AND TAUTOLOGY CHECKER 341
12.4 The LTL satisfiability and tautology checker
A formula φ ∈LTL(AP) is satisfiable if there is a Kripke structure A= (A,→A,L), with
L: A−→P(AP), and a computation path πsuch that A,π|= φ.Satisfiability of a formula φ∈
LTL(AP) isadecidableproperty. InMaude,thesatisfiabilitydecisionprocedureissupportedby
thefollowingpredefinedfunctionalmoduleSAT-SOLVER(alsointhefilemodel-checker.maude).
fmod SAT-SOLVER is
protecting LTL .
*** formula lists and results
sorts FormulaList SatSolveResult TautCheckResult .
subsort Formula < FormulaList .
subsort Bool < SatSolveResult TautCheckResult .
op nil : -> FormulaList [ctor] .
op _;_ : FormulaList FormulaList -> FormulaList
[ctor assoc id: nil] .
op model : FormulaList FormulaList -> SatSolveResult [ctor] .
op satSolve : Formula ~> SatSolveResult [special (...)] .
op counterexample :
FormulaList FormulaList -> TautCheckResult [ctor] .
op tautCheck : Formula ~> TautCheckResult .
op $invert : SatSolveResult -> TautCheckResult .
var F : Formula .
vars L C : FormulaList .
eq tautCheck(F) = $invert(satSolve(~ F)) .
eq $invert(false) = true .
eq $invert(model(L, C)) = counterexample(L, C) .
endfm
One can define the desired atomic predicates in a module extending SAT-SOLVER, such as,
for example,
fmod SAT-SOLVER-TEST is
extending SAT-SOLVER .
extending LTL .
ops a b c d e p q r : -> Formula .
endfm
The user can then decide the satisfiability of an LTL formula involving those atomic propo-
sitions by applying the operator satSolve (whose special attribute has also been omitted in
the module above) to the given formula and evaluating the expression. The resulting solution
of sort SatSolveResult is then either false, if no model exists, or a finite model satisfying
the formula. Such a model is described by a comma-separated pair of finite paths of states: an
initial path leading to a cycle. Each state is described by a conjunction of atomic propositions
or negated atomic propositions, with the propositions not mentioned in the conjunction being
“don’t care” ones. For example, we can evaluate
Maude> red satSolve(a /\ (O b) /\ (O O ((~ c) /\ [](c \/ (O c))))) .
reduce in SAT-SOLVER-TEST :
satSolve(O O (~ c /\ [](c \/ O c)) /\ (a /\ O b)) .
result SatSolveResult: model(a ; b, (~ c) ; c)

<!-- manual page 354 -->
342 CHAPTER 12. LTL MODEL CHECKING
a b ¬c c
Figure 12.2: Graphical representation of a Kripke structure
which is satisfied by a four-state model with a holding in the first state, b holding in the second,
c not holding in the third but holding in the fourth, and the fourth state going back to the
third, as shown in Figure 12.2.
We call φ∈LTL(AP) a tautology if and only if A,a|=LT L φholds for every Kripke structure
A= (A,→A,L) with L : A −→P(AP), and every state a ∈A. It then follows easily that φ
is a tautology if and only if¬φ is unsatisfiable. Therefore, the module SAT-SOLVER can also
be used as a tautology checker. This is accomplished by using the tautCheck, $invert, and
counterexample operators and their corresponding equations in SAT-SOLVER. The tautCheck
function returns either true if the formula is a tautology, or a finite model that does not satisfy
the formula. For example, we can evaluate:
Maude> red tautCheck((a => (O a)) <-> (a => ([] a))) .
reduce in SAT-SOLVER-TEST : tautCheck((a => O a) <-> a => []a) .
result Bool: true
Maude> red tautCheck((a -> (O a)) <-> (a -> ([] a))) .
reduce in SAT-SOLVER-TEST : tautCheck((a -> O a) <-> a -> []a) .
result TautCheckResult: counterexample(a ; a ; (~ a), True)
The tautology checker gives us also a decision procedure for semantic LTL equality, which
is further discussed in [53].
12.5 Model checking a round-robin scheduling algorithm
In this section we present a specification of a round-robin scheduling algorithm, and the mutual
exclusion and guaranteed reentrance properties are proven about it. Both the algorithm and
the property guaranteeing that all processes reenter their critical sections are parameterized by
the number of processes. We use Maude’s model checker to prove the mutual exclusion and
guaranteed reentrance properties.
This example, though quite simple, is interesting in several ways. The use of parameteri-
zation is exploited at both the system and the property level. At the system level, it allows
a succinct specification of a parametric system. At the property level, it makes possible the
specification of the relevant properties for each value of the parameter, also in a very succinct
way. This is quite useful, because the property formulas vary as the parameter changes, and
the parametric description encompasses an infinite number of instance formulas.
We first give a specification of natural numbers modulo. Since we want to be able to
have any number of processes, we define the NAT/ module parameterized by the functional
theory NZNAT#, which requires a constant of sort Nat. Thus, having a view, say 5 from TRIV
to NZNAT# mapping # to the natural number 5, the module expression NAT/{5} specifies the
natural numbers modulo 5.
fth NZNAT# is
protecting NAT .
op # : -> NzNat .
endfth

<!-- manual page 355 -->
12.5. MODEL CHECKING A ROUND-ROBIN SCHEDULING ALGORITHM 343
fmod NAT/{N :: NZNAT#} is
sort Nat/{N} .
op [_] : Nat -> Nat/{N} [ctor] .
op _+_ : Nat/{N} Nat/{N} -> Nat/{N} .
op _*_ : Nat/{N} Nat/{N} -> Nat/{N} .
vars N M : Nat .
ceq [N] = [N rem #] if N >= # .
eq [N] + [M] = [N + M] .
eq [N] * [M] = [N * M] .
endfm
The round-robin scheduling algorithm is specified in the module RROBIN below. Processes
are represented as objects of class Proc, which may be in wait or critical mode, meaning
that a process may be either in its critical section or waiting to enter into it. The process
getting the token, which is represented as the message go, can enter its critical section. Once
a process gets out of its critical section it forwards the token to the next process. The init
operator sets up the initial configuration for a given number of processes. Note that Nat/{N} is
made a subsort of Oid, making in this way natural numbers modulo N valid object identifiers.
omod RROBIN{N :: NZNAT#} is
protecting NAT/{N} .
sort Mode .
ops wait critical : -> Mode [ctor] .
subsort Nat/{N} < Oid .
class Proc | mode : Mode .
msg go : Nat/{N} -> Msg .
var N : Nat .
rl [enter] :
go([N])
< [N] : Proc | mode : wait >
=> < [N] : Proc | mode : critical > .
rl [exit] :
< [N] : Proc | mode : critical >
=> < [N] : Proc | mode : wait >
go([s(N)]) .
op init : -> Configuration .
op make-init : Nat/{N} -> Configuration .
ceq init = go([0]) make-init([N]) if s(N) := # .
ceq make-init([s(N)])
= < [s(N)] : Proc | mode : wait > make-init([N])
if N < # .
eq make-init([0]) = < [0] : Proc | mode : wait > .
endom
Forprovingmutualexclusionandguaranteedreentrance, wedeclarethepropositionsinCrit
and twoInCrit in the module CHECK-RROBIN below. The property inCrit takes a Nat/{N} as
argument, thus making this property parameterized by the number of processes, and is true

<!-- manual page 356 -->
344 CHAPTER 12. LTL MODEL CHECKING
whensuchaprocessisinitscriticalsection. ThepropertytwoInCritistrueifanytwoprocesses
are in their critical sections simultaneously. Mutual exclusion will be proved directly below,
while for proving guaranteed reentrance we use the auxiliary formula guaranteedReentrance,
which allows us to specify the property of all processes reentering their critical sections in
exactly 2N steps, for N the number of processes. For a formula F, nextIter F returns O...O
F (where O denotes the modal next operator), which specifies that the property is true in the
next iteration, that is, 2N steps later. Note that the expression 2 * # will become two times
N once the module is instantiated.
omod CHECK-RROBIN{N :: NZNAT#} is
pr RROBIN{N} .
inc MODEL-CHECKER .
inc SATISFACTION .
ex LTL-SIMPLIFIER .
inc LTL .
subsort Configuration < State .
op inCrit : Nat/{N} -> Prop .
op twoInCrit : -> Prop .
var N : Nat .
vars X Y : Nat/{N} .
var C : Configuration .
var F : Formula .
eq < X : Proc | mode : critical > C |= inCrit(X) = true .
eq < X : Proc | mode : critical > < Y : Proc | mode : critical > C
|= twoInCrit = true .
op guaranteedReentrance : -> Formula .
op allProcessesReenter : Nat -> Formula .
op nextIter_ : Formula -> Formula .
op nextIterAux : Nat Formula -> Formula .
eq guaranteedReentrance = allProcessesReenter(#) .
eq allProcessesReenter(s N)
= (inCrit([s N]) -> nextIter inCrit([s N])) /\
allProcessesReenter(N) .
eq allProcessesReenter(0) = inCrit([0]) -> nextIter inCrit([0]) .
eq nextIter F = nextIterAux(2 * #, F) .
eq nextIterAux(s N, F) = O nextIterAux(N, F) .
eq nextIterAux(0, F) = F .
endom
Note that the LTL formula describing the guaranteedReentrance property is not a single
LTL formula, but an infinite parametric family of formulas
guaranteedReentrance= {allProcessesReenter(n) |n∈N}.
The use of equations in the above CHECK-RROBIN parameterized module allows us to define this
infinite family of formulas by means of a few recursive equations. When this module is instanti-

<!-- manual page 357 -->
12.5. MODEL CHECKING A ROUND-ROBIN SCHEDULING ALGORITHM 345
atedforaconcretevalueofn,wethenobtaintheconcreteLTLformulaallProcessesReenter(n)
for that n.
We now prove mutual exclusion and guaranteed reentrance for the case of five processes
using the model checker.
view 5 from NZNAT# to NAT is
op # to term 5 .
endv
Maude> reduce in CHECK-RROBIN{5} :
modelCheck(init, [] ~ twoInCrit) .
result Bool: true
Maude> reduce in CHECK-RROBIN{5} :
modelCheck(init, [] guaranteedReentrance) .
result Bool: true
Ofcoursetheanswerdependsonthepropertycheckedandisnotalwaystrue. Thefollowing
example shows how the model checker gives a counterexample as result when trying to prove
that, for a configuration of five processes, process [1] is in its critical section three steps after
it was in it.
Maude> red in CHECK-RROBIN{5} :
modelCheck(init, [] (inCrit([1]) -> O O O inCrit([1]))) .
result ModelCheckResult:
counterexample(
{ go([0])
< [0] : Proc | mode : wait > < [1] : Proc | mode : wait >
< [2] : Proc | mode : wait > < [3] : Proc | mode : wait >
< [4] : Proc | mode : wait >, ’enter }
{ < [0] : Proc | mode : critical > < [1] : Proc | mode : wait >
< [2] : Proc | mode : wait > < [3] : Proc | mode : wait >
< [4] : Proc | mode : wait >, ’exit }
{ go([1])
< [0] : Proc | mode : wait > < [1] : Proc | mode : wait >
< [2] : Proc | mode : wait > < [3] : Proc | mode : wait >
< [4] : Proc | mode : wait >, ’enter }
{ < [0] : Proc | mode : wait > < [1] : Proc | mode : critical >
< [2] : Proc | mode : wait > < [3] : Proc | mode : wait >
< [4] : Proc | mode : wait >, ’exit }
{ go([2])
< [0] : Proc | mode : wait > < [1] : Proc | mode : wait >
< [2] : Proc | mode : wait > < [3] : Proc | mode : wait >
< [4] : Proc | mode : wait >, ’enter }
{ < [0] : Proc | mode : wait > < [1] : Proc | mode : wait >
< [2] : Proc | mode : critical > < [3] : Proc | mode : wait >
< [4] : Proc | mode : wait >, ’exit },
{ go([3])
< [0] : Proc | mode : wait > < [1] : Proc | mode : wait >
< [2] : Proc | mode : wait > < [3] : Proc | mode : wait >
< [4] : Proc | mode : wait >,’enter }
{ < [0] : Proc | mode : wait > < [1] : Proc | mode : wait >
< [2] : Proc | mode : wait > < [3] : Proc | mode : critical >
< [4] : Proc | mode : wait >, ’exit }
{ go([4])

<!-- manual page 358 -->
346 CHAPTER 12. LTL MODEL CHECKING
< [0] : Proc | mode : wait > < [1] : Proc | mode : wait >
< [2] : Proc | mode : wait > < [3] : Proc | mode : wait >
< [4] : Proc | mode : wait >, ’enter }
{ < [0] : Proc | mode : wait > < [1] : Proc | mode : wait >
< [2] : Proc | mode : wait > < [3] : Proc | mode : wait >
< [4] : Proc | mode : critical >, ’exit }
{ go([0])
< [0] : Proc | mode : wait > < [1] : Proc | mode : wait >
< [2] : Proc | mode : wait > < [3] : Proc | mode : wait >
< [4] : Proc | mode : wait >,’enter}
{ < [0] : Proc | mode : critical > < [1] : Proc | mode : wait >
< [2] : Proc | mode : wait > < [3] : Proc | mode : wait >
< [4] : Proc | mode : wait >,’exit}
{ go([1])
< [0] : Proc | mode : wait > < [1] : Proc | mode : wait >
< [2] : Proc | mode : wait > < [3] : Proc | mode : wait >
< [4] : Proc | mode : wait >,’enter}
{ < [0] : Proc | mode : wait > < [1] : Proc | mode : critical >
< [2] : Proc | mode : wait > < [3] : Proc | mode : wait >
< [4] : Proc | mode : wait >,’exit}
{ go([2])
< [0] : Proc | mode : wait > < [1] : Proc | mode : wait >
< [2] : Proc | mode : wait > < [3] : Proc | mode : wait >
< [4] : Proc | mode : wait >, ’enter }
{ < [0] : Proc | mode : wait > < [1] : Proc | mode : wait >
< [2] : Proc | mode : critical > < [3] : Proc | mode : wait >
< [ 4] : Proc | mode : wait >,’exit } )
In [24, Section 16.6] some properties of a Mobile Maude application are model checked. This
example is interesting because two levels of reflection (see Chapter 17) are involved: the object
level, at which Mobile Maude system code executes, and the metalevel, at which application
code is executed.
12.6 LTL+ formulas and the LTL+ model checker
An LTL formula φis a property that (tacitly) is universally quantified over all the computation
paths π starting at an initial state init. That is, if M is a Maude system module in which the
state predicates appearing in φhave been equationally defined and init is an initial state in M,
thenM,init |=LTL φholdsiffforallcomputationpathsπofM startingatinit, π|=LTL φholds
(where we have left implicit the Kripke structure associated to M). In a logic like CTL* [19]
such a universal path quantifier would be made explicit, so that the LTL formula φ would
be written as the universally path quantified formula A φ. There are useful properties of a
concurrent system not expressible in LTL itself.
Consider, for example, the following object-oriented module specifying a simple communi-
cation protocol in Maude:
fmod ANOTHER-NAT-LIST is
protecting NAT .
sort List .
subsorts Nat < List .
op nil : -> List .
op _;_ : List List -> List [assoc id: nil] .
op |_| : List -> Nat . *** length function

<!-- manual page 359 -->
12.6. LTL+ FORMULAS AND THE LTL+ MODEL CHECKER 347
var N : Nat . var L : List .
eq | nil | = 0 .
eq | N ; L | = s(| L |) .
endfm
omod COMM-PROTOCOL is
protecting ANOTHER-NAT-LIST .
protecting QID .
subsort Qid < Oid .
class Sender | buff : List, rec : Oid, cnt : Nat, ack-w : Bool .
class Receiver | buff : List, snd : Oid, cnt : Nat .
msg to_from_val_cnt_ : Oid Oid Nat Nat -> Msg .
msg to_from_ack_ : Oid Oid Nat -> Msg .
op init : Oid Oid List -> Configuration .
vars N M : Nat .
vars L Q : List .
vars A B : Oid .
var TV : Bool .
eq init(A,B,L)
= < A : Sender | buff : L, rec : B, cnt : 0, ack-w : false >
< B : Receiver | buff : nil, snd : A, cnt : 0 > .
rl [snd] :
< A : Sender | buff : (N ; L), rec : B, cnt : M, ack-w : false >
=> (to B from A val N cnt M)
< A : Sender | buff : L, cnt : M, ack-w : true > .
rl [rec] :
< B : Receiver | buff : L, snd : A, cnt : M >
(to B from A val N cnt M)
=> (to A from B ack M)
< B : Receiver | buff : (L ; N), snd : A, cnt : s(M) > .
rl [ack-rec] :
(to A from B ack M)
< A : Sender | buff : L, rec : B, cnt : M, ack-w : true >
=> < A : Sender | buff : L, rec : B, cnt : s(M), ack-w : false > .
endom
There are two classes, for Senders and for Receivers, and two messages, for sending a value
and for sending an acknowledgement. The equation defines an initial state with one Sender
and one Receiver; the first two arguments of the init operator provide the respective names
for those two objects, while the third argument provides the list of values that are going to be
sent.
There are three rules: snd creates a message to send a value, rec puts the value from
the message into the receiver’s buffer while creating an acknowledgement message, and finally
ack-rec handles this last message.
We could ask the following question about this protocol:
Are there states reachable from init(A,B,L) such that the counters of A and B hold
different values?

<!-- manual page 360 -->
348 CHAPTER 12. LTL MODEL CHECKING
This property about COMM-PROTOCOL cannot be expressed in LTL, but has a simple expres-
sion as a formula existentially quantified over paths, namely, the formula:
EQ¬same.counters(A,B),
where the same.counters(A,B) predicate is defined as follows:
omod COMM-PROTOCOL-PREDS is
protecting COMM-PROTOCOL .
including SATISFACTION .
subsort Configuration < State .
op same.cnts : Oid Oid -> Prop .
vars N M : Nat .
vars L1 L2 : List .
vars A B : Oid .
var TV : Bool .
var C : Configuration .
eq < A : Sender | buff : L2, rec : B, cnt : N, ack-w : TV >
< B : Receiver | buff : L1, snd : A, cnt : M > C |= same.cnts(A,B)
= N == M .
endom
By definition, COMM-PROTOCOL-PREDS,init(A,B,L) |=LTL+ EQ¬same.counters(A,B) holds iff
there is a computation path π in COMM-PROTOCOL-PREDS starting at init(A,B,L) such that
π|=LTL Q¬same.counters(A,B)
holds (where we have again left implicit the associated Kripke structure).
Two points should be noted about the above definitional equivalence: (i) since our formula
is not an LTL formula but belongs to an extended temporal logic that we shall call LTL+, the
semantic satisfaction relation is qualified as |=LTL+; and (ii) in such a definitional extension
the path satisfaction relation used is |=LTL. That is, the meaning of our LTL+ formula can
be expressed in terms of the LTL path satisfaction relation. What is LTL+? In Maude it is
defined as the following functional module (in the file model-checker.maude).
fmod LTL+ is
protecting LTL .
sort Formula+ .
op E_ : Formula -> Formula+ [ctor] .
op A_ : Formula -> Formula+ [ctor] .
op ~_ : Formula+ -> Formula+ .
var f : Formula .
eq ~(E f) = A (~ f) .
eq ~(A f) = E (~ f) .
endfm
That is, an LTL+ formula has one of two forms: it is either an existentially path quantified
formula E φ or a universally path quantified formula A φ, with φ an LTL formula. LTL+
formulas are closed under negation as expressed by the two equations in LTL+.
Using the duality of path quantifiers E φ≡¬A¬φ, we have the following two definitional
equivalences that define the semantics of LTL+ formulas in terms of the semantics of LTL

<!-- manual page 361 -->
12.6. LTL+ FORMULAS AND THE LTL+ MODEL CHECKER 349
formulas:
M,init |=LTL+ A φ ⇔ M,init |=LTL φ
M,init |=LTL+ E φ ⇔ M,init ̸|=LTL ¬φ
These equivalences give us an easy way of extending Maude’s LTL model checker to an LTL+
model checker, which is defined in Maude (also in the file model-checker.maude) as follows:
fmod MODEL-CHECKER+ is
protecting MODEL-CHECKER .
including LTL+ .
sort ModelCheck+Result .
subsort ModelCheckResult < ModelCheck+Result .
op witness : TransitionList TransitionList -> ModelCheck+Result [ctor] .
op modelCheck+ : State Formula+ ~> ModelCheck+Result .
op neg : ModelCheck+Result -> ModelCheck+Result .
var f : Formula . var st : State .
vars TL1 TL2 : TransitionList .
eq modelCheck+(st, (E f)) = neg(modelCheck(st, (~ f))) .
eq modelCheck+(st, (A f)) = modelCheck(st, f) .
eq neg(true) = false .
eq neg(false) = true .
eq neg(counterexample(TL1, TL2)) = witness(TL1, TL2) .
eq neg(witness(TL1, TL2)) = counterexample(TL1, TL2) .
endfm
Using MODEL-CHECKER+ we can answer our original question about whether in some states
reachable from init(A,B,L) the values in the counters of A and B differ. We can do so by
loading the module:
omod COMM-PROTOCOL-CHECK is
protecting COMM-PROTOCOL-PREDS .
including MODEL-CHECKER+ .
endom
and then giving the command
Maude> red in COMM-PROTOCOL-CHECK :
modelCheck+(init(’a, ’b, 1 ; 2 ; 3), E (<> ~ same.cnts(’a, ’b))) .
result ModelCheck+Result:
witness(
{< ’a : Sender | buff : (1 ; 2 ; 3), rec : ’b,
cnt : 0, ack-w : false >
< ’b : Receiver | buff : nil, cnt : 0, snd :’a >, ’snd}
{< ’a : Sender | buff : (2 ; 3), rec : ’b, cnt : 0, ack-w : true >
< ’b : Receiver | buff : nil, cnt : 0, snd : ’a >
to ’b from ’a val 1 cnt 0, ’rec}
{< ’a : Sender | buff : (2 ; 3), rec : ’b, cnt : 0, ack-w : true >
< ’b : Receiver | buff : 1, cnt : 1, snd : ’a >
to ’a from ’b ack 0, ’ack-rec}
{< ’a : Sender | buff : (2 ; 3), rec : ’b, cnt : 1, ack-w : false >
< ’b : Receiver | buff : 1, cnt : 1, snd : ’a >, ’snd}

<!-- manual page 362 -->
350 CHAPTER 12. LTL MODEL CHECKING
{< ’a : Sender | buff : 3, rec : ’b, cnt : 1, ack-w : true >
< ’b : Receiver | buff : 1, cnt : 1, snd : ’a >
to ’b from ’a val 2 cnt 1, ’rec}
{< ’a : Sender | buff : 3, rec : ’b, cnt : 1, ack-w : true >
< ’b : Receiver | buff : (1 ; 2), cnt : 2, snd : ’a >
to ’a from ’b ack 1, ’ack-rec}
{< ’a : Sender | buff : 3, rec : ’b, cnt : 2, ack-w : false >
< ’b : Receiver | buff : (1 ; 2), cnt : 2, snd : ’a >, ’snd}
{< ’a : Sender | buff : nil, rec : ’b, cnt : 2, ack-w : true >
< ’b : Receiver | buff : (1 ; 2), cnt : 2, snd : ’a >
to ’b from ’a val 3 cnt 2, ’rec}
{< ’a : Sender | buff : nil, rec : ’b, cnt : 2, ack-w : true >
< ’b : Receiver | buff : (1 ; 2 ; 3), cnt : 3, snd : ’a >
to ’a from ’b ack 2, ’ack-rec},
{< ’a : Sender | buff : nil, rec : ’b, cnt : 3, ack-w : false >
< ’b : Receiver | buff : (1 ; 2 ; 3), cnt : 3, snd : ’a >, deadlock})
Thatis,theLTL+modelchekerverifiesthatourLTL+propertyE (<> ~ same.cnts( ’a, ’b))
holds from the initial state init(’a, ’b, 1 ; 2 ; 3) in a constructive way, i.e., by providing
a witness infinite path showing states where the values of the two counters differ.
Note that the equations
eq modelCheck+(st, (E f)) = neg(modelCheck(st, (~ f))) .
eq neg(witness(TL1, TL2)) = counterexample(TL1, TL2) .
in MODEL-CHECKER+ identify a witness for an LTL+ formula Eφwith a counterexample for the
negated LTL formula¬φ. For example, the above witness proving the LTL+ formula
EQ¬same.counters(A,B),
from the initial state init(’a, ’b, 1 ; 2 ; 3) exactly corresponds to a counterexample for
the negated LTL formula
¬Q¬same.counters(A,B),
from the same initial state, i.e., a counterexample for the equivalent LTL formula
Psame.counters(A,B),
stating that same.counters(A,B) is an invariant from that initial state. Indeed, we get:
Maude> red in COMM-PROTOCOL-CHECK :
modelCheck(init(’a,’b,(1 ; 2 ; 3)),([] same.cnts(’a,’b))) .
result ModelCheckResult:
counterexample(
{< ’a : Sender | buff : (1 ; 2 ; 3), rec : ’b, cnt : 0, ack-w : false >
< ’b : Receiver | buff : nil, cnt : 0, snd : ’a >, ’snd}
{< ’a : Sender | buff : (2 ; 3), rec : ’b, cnt : 0, ack-w : true >
< ’b : Receiver | buff : nil, cnt : 0, snd : ’a >
to ’b from ’a val 1 cnt 0, ’rec}
{< ’a : Sender | buff : (2 ; 3), rec : ’b, cnt : 0, ack-w : true >
< ’b : Receiver | buff : 1, cnt : 1, snd : ’a >
to ’a from ’b ack 0, ’ack-rec}
{< ’a : Sender | buff : (2 ; 3), rec : ’b, cnt : 1, ack-w : false >
< ’b : Receiver | buff : 1, cnt : 1, snd : ’a >, ’snd}
{< ’a : Sender | buff : 3, rec : ’b, cnt : 1, ack-w : true >
< ’b : Receiver | buff : 1, cnt : 1, snd : ’a >

<!-- manual page 363 -->
12.6. LTL+ FORMULAS AND THE LTL+ MODEL CHECKER 351
to ’b from ’a val 2 cnt 1, ’rec}
{< ’a : Sender | buff : 3, rec : ’b, cnt : 1, ack-w : true >
< ’b : Receiver | buff : (1 ; 2), cnt : 2, snd : ’a >
to ’a from ’b ack 1, ’ack-rec}
{< ’a : Sender | buff : 3, rec : ’b, cnt : 2, ack-w : false >
< ’b : Receiver | buff : (1 ; 2), cnt : 2, snd : ’a >, ’snd}
{< ’a : Sender | buff : nil, rec : ’b, cnt : 2, ack-w : true >
< ’b : Receiver | buff : (1 ; 2), cnt : 2, snd : ’a >
to ’b from ’a val 3 cnt 2, ’rec}
{< ’a : Sender | buff : nil, rec : ’b, cnt : 2, ack-w : true >
< ’b : Receiver | buff : (1 ; 2 ; 3), cnt : 3, snd : ’a >
to ’a from ’b ack 2, ’ack-rec},
{< ’a : Sender | buff : nil, rec : ’b, cnt : 3, ack-w : false >
< ’b : Receiver | buff : (1 ; 2 ; 3), cnt : 3, snd : ’a >, deadlock})
I.e., the same computation path is, simultaneously, a witness for the truth of the LTL+ formula
EQ¬same.counters(A,B) and a counterexample for the LTL formula Psame.counters(A,B)
from the same initial state init(’a, ’b, 1 ; 2 ; 3).

<!-- manual page 364 -->
352 CHAPTER 12. LTL MODEL CHECKING

