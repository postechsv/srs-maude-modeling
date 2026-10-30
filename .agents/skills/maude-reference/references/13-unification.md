# Chapter 13: Unification (manual pages 365-382)

<!-- manual page 365 -->
Chapter 13
Unification
13.1 Introduction
Unification is the solving of equations, either in free algebras of the form TΣ(X), or in relatively
free algebras modulo a set E of equations, that is, in algebras of the form TΣ/E(X). The first
caseissometimes calledsyntactic unification. ThesecondcaseissometimescalledE-unification
or unification modulo E; if E is not explicitly mentioned, then it is called equational unification
or semantic unification.
In solving any equation, such as, for example,
f(x,h(y)) = f(g(y),z)
we look for instances of the variables appearing in the equation that make both sides equal.
Variables can of course be instantiated by substitutions. A substitution that makes both sides of
the equation equal, that is, a solution of the equation, is called a unifier. For example, if we are
solving the above equation in the free algebra TΣ(X) with X a countable set of variables and
with Σ having a single sort (unsorted unification), the substitution σ = {x→g(y),z →h(y)}
is a unifier, and indeed the so-called most general unifier, so that for any other unifier ρ there
exists a substitution µ such that ρ= σ; µ, where σ; µ denotes composition of substitutions in
diagrammatic order. That is, any other solution of the equation is an instance of the most
general solution provided by σ.
Of course, some equations may not have syntactic unifiers, but may have semantic unifiers
modulo some equations E. Consider, for example, the equation
f(h(y),x) = f(g(y),z)
which obviously does not have any solution in TΣ(X). It does, however, have a solution in
TΣ/C(X), where C is the commutativity axiom f(x,y) = f(y,x). Indeed, the exact same
substitution σ solving the first equation f(x,h(y)) = f(g(y),z) in a syntactic way, is now
a unifier solving the second equation f(h(y),x) = f(g(y),z) modulo C, because we have
f(h(y),g(y)) =C f(g(y),h(y)).
Unificationisafundamentaldeductivemechanismusedinmanyautomateddeductiontasks.
It is also very important in combining the paradigms of functional programming and logic
programming (in the Prolog sense of “logic programming”). Furthermore, in the context of
Maude, unification can be very useful to reason not only about equational theories (functional
modules or theories), but also about rewrite theories (system modules or theories) [100].
353

<!-- manual page 366 -->
354 CHAPTER 13. UNIFICATION
Therefore, it is very useful to have an efficient implementation of unification available in
Maude, which is what this chapter describes. Specifically, we explain how order-sorted unifica-
tion modulo frequently occurring equational axioms is supported in Maude. In Chapter 14, we
explainorder-sortedunificationmoduloconvergentequationaltheories, alsoavailableinMaude.
13.2 Order-sorted unification
Although the most general equational theories supported by Maude are membership equational
theories, to obtain practical unification algorithms, allowing us to effectively compute the solu-
tions of an equational unification problem, it is important to restrict ourselves to order-sorted
equational theories. We can define the basic concepts of order-sorted E-unification in full
generality.
Given an order-sorted equational theory (Σ,E), an E-unification problem consists of a
nonempty set of unification equations of the form t=? t′, written in the notation
t1 =
? t′
1 ∧... ∧tn =
? t′
n
where n≥1 and the “conjunction” operator ∧is assumed to be associative and commutative.
Given such an E-unification problem, an E-unifier for it is an order-sorted substitution1
σ : Vars(t1,t′
1,...,tn,t′
n) −→TΣ(X) (where we assume that the set X of variables contains a
countable number of variables for each sort) such that, for all i= 1,...,n,
E ⊢(∀Yi) σ(ti) = σ(t′
i),
where Yi = Vars(σ(ti),σ(t′
i)), that is, all the equations (∀Yi) σ(ti) = σ(t′
i) can be deduced in
(membership) equational logic from the set of equations E.
A set Uof unifiers is called a complete set of E-unifiers for a given E-unification problem
t1 =? t′
1 ∧...∧tn =? t′
n iff for any other E-unifier ρ of the same E-unification problem there
exists a substitution µ and a unifier σ ∈Usuch that ρ=E σ; µ, that is, for each variable x in
the domain of ρwe have E ⊢ρ(x) = µ(σ(x)). A complete set Uof E-unifiers is called minimal
if any proper subset of Ufails to be complete.
For an order-sorted equational theory (Σ,E), unification is said to be finitary if for any
E-unification problem there is always a finite complete set Uof unifiers. Similarly, unification
for (Σ,E) is called unitary if it is finitary and for any E-unification problem a minimal complete
set of unifiers is always either empty or a singleton set. We say that (Σ,E) has a unification
algorithm if there is an algorithm generating a complete set of E-unifiers for any E-unification
problem in (Σ,E).
Unlike unsorted syntactic unification, which always either fails or has a single most general
unifier, order-sorted syntactic unification is not necessarily unitary, that is, there is in general
no single most general unifier. What exists (if Σ is finite) is a finite minimal complete set of
syntactic unifiers. For some commonly occurring theories having a unification algorithm, such
as the theory Aof associativity of a binary function symbol, it is well-known that unification is
not finitary: in general an infinite number of solutions may exist (see Section 13.4.6). However,
for other theories, such as commutativity C or associativity-commutativity AC, unification is
finitary, both when Σ is unsorted and when Σ is order-sorted (and finite).
1That is, a sort-preserving mapping from Vars(t1,t′
1,...,tn,t′
n), the set of all variables appearing in the
terms ti or t′
i, to TΣ(X).

<!-- manual page 367 -->
13.3. THEORIES CURRENTLY SUPPORTED 355
13.2.1 A hybrid approach to equational order-sorted unification
We define a hybrid approach, in which we take advantage of Maude’s structuring of a mod-
ule’s equations into equational axioms Ax, such as associativity, and/or commutativity, and/or
identity, and equations E, which are assumed to be coherent,2 confluent, and terminating
modulo Ax. We can then consider order-sorted equational theories of the form (Σ,E ∪Ax)
and decompose the E∪Ax-unification problem into two problems: one of Ax-unification, and
another of E ∪Ax-unification that uses an Ax-unification algorithm as a subroutine. The
point of this decomposition is that Ax-unification needs to be very efficient in order to have
E ∪Ax-unification be built on top of Ax-unification. Since the axioms Ax are well-known
and unification algorithms exist for them, the task of building in efficient Ax-unification algo-
rithms, although highly nontrivial, becomes manageable; which is what this chapter describes.
Chapter 14 describes such E∪Ax-unification relying on this Ax-unification.
13.3 Theories currently supported
As mentioned above, a practical way of dealing with order-sorted equational unification is to
consider order-sorted theories of the form (Σ,E∪Ax), with Ax a set of commonly occurring
axioms, declared in Maude as equational attributes (see Section 4.4.1), and E the remaining
equations specified with the eqor ceqkeywords. We can then decompose the E∪Ax-unification
problem into an Ax-unification problem and an E∪Ax-unification problem that uses an Ax-
unification algorithm as a subroutine. In such a decomposition, the efficiency of the Ax-
unification algorithm becomes crucial.
Maude currently provides an order-sorted Ax-unification algorithm for all order-sorted theo-
ries (Σ,E∪Ax) such that the order-sorted signature Σ is preregular modulo Ax(see Sections 3.8
and 21.3.5) and the axioms Ax associated to function symbols are as follows:
• there can be arbitrary function symbols and constants with no equational attributes;
• the iter equational attribute can be declared for some unary symbols;
• different equational attributes can be declared for some binary function symbols:
– the assoc attribute3; usually referred to as A (for associative),
– the comm attribute; usually referred to as C (for commutative),
– the assoc comm attributes; usually referred to as AC (for associative and commu-
tative),
– the assoc comm id: attributes; usually referred to as ACU (for associative and
commutative with identity, or unit),
– the assoc id: attributes3; usually referred to as AU (for associative and identity,
or unit),
– the comm id: attributes; usually referred to as CU (for commutative and identity,
or unit),
– the id: attribute; usually referred to as U (for identity, or unit),
2Coherence of E modulo Ax is very closely related to the notion of coherence of rules relative to equations
explained in Section 5.3; see [75, 96] for the precise definition of coherence in the equational case. The main role
of coherence of E modulo Ax is to get the effect of rewriting in Ax-equivalence classes with E. For example,
for Ax= AC, coherence modulo AC is easily achieved by adding to each equation in E with a top AC function
symbol in its lefthand side a similar equation with an extra “extension variable” argument added to the AC
function symbol, as explained in Section 4.8. Section 4.8 also explains how, for rewriting modulo axioms Ax
supported by Maude, Maude automatically performs such a coherence completion in an implicit, built-in way
using extension variables and “extension aware” Ax-matching algorithms.
3Although unification with the assoc attribute but without the comm attribute is infinitary in general, we
support a substantial class of equational problems for which it is finitary, and provide a finite but incomplete
set of solutions (with an explicit warning that it may be incomplete) in all other cases. See Section 13.4.6.

<!-- manual page 368 -->
356 CHAPTER 13. UNIFICATION
– the left id: attribute; usually referred to as Ul (for left identity, or unit), and
– the right id: attribute; usually referred to as Ur (for right identity, or unit),
but no other equational attributes must be given for such symbols.
Explicitly excluded are theories with a binary symbol declared with the idem attribute
and associativity with only left or right identity. However, these cases have the finite variant
property, so that, used as variant equations, they can be combined with all the other just-
mentioned attributes by means of variant unification algorithms (see Section 14.8).
If we give to Maude a unification problem in a functional module of the form fmod (Σ,Ax)
endfm where Σ and Ax satisfy the above requirements, we get a complete4 set of order-sorted
unifiers modulo the theory (Σ,Ax). If, instead, we give the same problem to Maude in the
functional module fmod (Σ,E∪Ax) endfm, then the equations E are ignored, and we get the
same unifiers as for the module or theory obtained by omitting the equations E. Similarly, if we
provide the same unification problem in a functional theory fth (Σ,E∪Ax) endfth, a system
module mod (Σ,E∪Ax,R) endm or a system theory th (Σ,E∪Ax,R) endth, we again get the
same set of unifiers as for the theory (Σ,Ax). All this is consistent with the decomposition
idea mentioned above: to deal with order-sorted E∪Ax-unification, other methods, that use
the Ax-unification algorithm as a component, can later be defined.
Maude is even more tolerant than this. The user can give to Maude a unification problem in
afunctionalmodule(orfunctionaltheory, orsystemmodule, orsystemtheory)oftheformfmod
(Σ,E∪M∪Ax∪Ax′) endfm (or the analogous specification in the other cases), where (Σ,Ax)
satisfies the conditions mentioned above, but M is an optional set of membership axioms (that
is, (Σ,E∪M∪Ax∪Ax′) can be a membership equational theory and not just an order-sorted
theory), and the axioms Ax′violate those conditions mentioned above. Then what will happen
is:
1. As before, the additional equations E (or the rules R of a system module or theory) are
completely ignored, and the membership axioms M are likewise ignored.
2. If a unification problem involves the occurrence of a symbol satisfying axioms Ax′at the
root position of a non-ground subterm, the unification process will fail and a warning will
be printed (see Section 13.4.1).
3. If a unification problem involves the occurrence of symbols satisfying axioms Ax′, but all
such occurrences are always in ground subterms of the problem, then this special case of
Ax∪Ax′-unification is handled by Maude and the corresponding Ax∪Ax′-unifiers are
returned (see Section 13.4.1).
Furthermore, the functional module fmod (Σ,E∪M ∪Ax∪Ax′) endfm (or the analogous
functional theory or system module or theory) may import predefined modules such as BOOL
or NAT, so that function symbols in such predefined modules can also be used in unification
problems.
13.4 The unify command
Given a functional module or theory, or a system module or theory, ⟨ModId⟩, the user can give
to Maude a unification command of the following two forms:
unify [ n ] in ⟨ModId⟩ :
⟨Term-1⟩ =? ⟨Term’-1⟩ /\ ... /\ ⟨Term-k⟩ =? ⟨Term’-k⟩.
irredundant unify [ n ] in ⟨ModId⟩ :
⟨Term-1⟩ =? ⟨Term’-1⟩ /\ ... /\ ⟨Term-k⟩ =? ⟨Term’-k⟩.
4Or a finite but incomplete set (with an explicit warning) when one of the symbols has the assoc attribute
without the comm attribute and the equations fall outside the class for which a finite, complete set of unifiers
can be computed.

<!-- manual page 369 -->
13.4. THE UNIFY COMMAND 357
where k≥1; nis an optional argument providing a bound on the number of unifiers requested,
so that if the cardinality of the set of unifiers is greater than the specified bound, the unifiers
beyond that bound are omitted; and ⟨ModId⟩can be any module or theory declared in the
current Maude session (as usual, if no module is mentioned, the current module is used). The
second command generates all the unifiers and, then, filters them against each other in order
to return a minimal set of most general unifiers modulo the axioms.
For a simple example of syntactic order-sorted unification problem illustrating:
• the use of the unify command;
• sort downgrading of a variable;
• the use of the predefined operator _^_ in the NAT module (see Section 8.3), representing
exponentiation on natural numbers; and
• the, in general, non-unitary nature of order-sorted unification,
we can define the module
fmod UNIFICATION-EX1 is
extending NAT .
op f : Nat Nat -> Nat .
op f : NzNat Nat -> NzNat .
op f : Nat NzNat -> NzNat .
endfm
and then give to Maude the following command:
Maude> unify f(X:Nat, Y:Nat) ^ B:NzNat =? A:NzNat ^ f(Y:Nat, Z:Nat) .
Unifier 1
X:Nat --> #1:Nat
Y:Nat --> #2:NzNat
B:NzNat --> f(#2:NzNat, #3:Nat)
A:NzNat --> f(#1:Nat, #2:NzNat)
Z:Nat --> #3:Nat
Unifier 2
X:Nat --> #1:NzNat
Y:Nat --> #2:Nat
B:NzNat --> f(#2:Nat, #3:NzNat)
A:NzNat --> f(#1:NzNat, #2:Nat)
Z:Nat --> #3:NzNat
The next example in the same module illustrates the use of the unify command with a
unification problem consisting of two equations:
Maude> unify f(X:Nat, Y:NzNat) =? f(Z:NzNat, U:Nat)
/\ V:NzNat =? f(X:Nat, U:Nat) .
Unifier 1
X:Nat --> #1:NzNat
Y:NzNat --> #2:NzNat
Z:NzNat --> #1:NzNat
U:Nat --> #2:NzNat
V:NzNat --> f(#1:NzNat, #2:NzNat)
Notethat, asalreadymentioned, wecouldinsteadinvoketheunifycommandinafunctional
or system module or theory having additional equations, memberships, or rules, which are
always ignored. For example, we could have instead declared the system theory

<!-- manual page 370 -->
358 CHAPTER 13. UNIFICATION
th UNIFICATION-EX2 is
including NAT .
op f : Nat Nat -> Nat .
op f : NzNat Nat -> NzNat .
op f : Nat NzNat -> NzNat .
eq f(f(N:Nat, M:Nat), K:Nat) = f(N:Nat, M:Nat) .
rl f(N:Nat, M:Nat) => 0 .
endth
so that, if we give again the same unify commands above, we will obtain exactly the same sets
of order-sorted unifiers as for the UNIFICATION-EX1 module.
The following unification command in the predefined CONVERSION module (see Section 8.10)
illustratesafurtherpointonthehandlingofbuilt-inconstantsandfunctions. Built-inconstants,
even though infinite in number (all strings, all quoted identifiers, all natural numbers, and so
on), are handled and can be used in unification problems. But built-in functions are not
considered built-in for unification purposes; therefore, built-in evaluation of such functions is
not performed during the unification.
Maude> unify in CONVERSION :
X:String < "foo" + Y:Char =?
Z:String + string(pi) < "foo" + Z:String .
Unifier 1
X:String --> #1:Char + string(pi)
Y:Char --> #1:Char
Z:String --> #1:Char
The above examples illustrate a further point about the form of the returned unifiers,
namely, that in each assignment X –> t in a unifier, the variables appearing in the term t are
always fresh variables of the form #n:Sort. The user is required5 not to use variables of this
form in the submitted unification problem. A warning is printed if this requirement is violated:
Maude> unify in NAT : X:Nat ^ #1:Nat =? #2:Nat .
Warning: unsafe variable name #1:Nat in unification problem.
13.4.1 Non-supported unification examples
The handling of unification problems in modules with operators whose equational axioms are
excluded from the current unification algorithm can be illustrated by means of the following
module:
fmod UNIFICATION-EX3 is
extending NAT .
op f : Nat Nat -> Nat [idem] .
endfm
As already mentioned, a unification problem using such an idempotent function symbol f in a
non-ground subterm will mean that the unification attempt fails and a warning is issued:
Maude> unify f(f(X:Nat, Y:Nat), Z:Nat) =? f(A:Nat, B:Nat) .
Warning: Term f(X:Nat, Y:Nat, Z:Nat) is non-ground and unification
for its top symbol is not currently supported.
5When unification is used at the metalevel (see Section 17.6.8), there are two possibilities: (i) a counter for
new fresh variables must take into account the numbers used in the variables of the unification problem, or (ii)
an identifier among ’#, ’%, and ’@ used in the variables of the unification problem.

<!-- manual page 371 -->
13.4. THE UNIFY COMMAND 359
Instead, if all symbols satisfying unsupported equational axioms Ax′ only appear in ground
subterms of the unification problem, the unification attempt succeeds with the correct set of
order-sorted Ax∪Ax′-unifiers:
Maude> unify X:Nat + f(f(41, 42),43) =? Y:Nat + f(41,f(42,43)) .
Unifier 1
X:Nat --> #1:Nat + f(41, f(42, 43))
Y:Nat --> #1:Nat + f(f(41, 42), 43)
Unifier 2
X:Nat --> f(41, f(42, 43))
Y:Nat --> f(f(41, 42), 43)
Note, however, that, as already mentioned, unification modulo the idem attribute can be
achievedbyvariant-basedunificationusingtheexplicitequationf(N:Nat,N:Nat) = N:Nat(see
Section 14.8).
13.4.2 Associative-commutative (AC ) unification examples
The use of a bound on the number of unifiers, as well as the use of the associative-commutative
(AC) operator + in the predefined NAT module (see Section 8.3), plus the fact that even small
AC-unification problems can generate a large number of unifiers are all illustrated by the
following command:
Maude> unify [100] in NAT :
X:Nat + X:Nat + Y:Nat =? A:Nat + B:Nat + C:Nat .
Unifier 1
X:Nat --> #1:Nat + #2:Nat + #3:Nat + #5:Nat + #6:Nat + #8:Nat
Y:Nat --> #4:Nat + #7:Nat + #9:Nat
A:Nat --> #1:Nat + #1:Nat + #2:Nat + #3:Nat + #4:Nat
B:Nat --> #2:Nat + #5:Nat + #5:Nat + #6:Nat + #7:Nat
C:Nat --> #3:Nat + #6:Nat + #8:Nat + #8:Nat + #9:Nat
...
Unifier 100
X:Nat --> #1:Nat + #2:Nat + #3:Nat + #4:Nat
Y:Nat --> #5:Nat
A:Nat --> #1:Nat + #1:Nat + #2:Nat
B:Nat --> #2:Nat + #3:Nat
C:Nat --> #3:Nat + #4:Nat + #4:Nat + #5:Nat
The unification problem above has 381 unifiers, though only 100 were asked for. Indeed, it
is the minimal set of most general unifiers and the irredundant unify command returns the
same set with 381 unifiers.
13.4.3 Unification examples with the iter attribute
The following example illustrates the efficiency of order-sorted unification modulo the iter
theory (in this example in combination with the comm theory). Consider the following module:
fmod ITER-EXAMPLE is
sorts NzEvenNat EvenNat OddNat NzNat Nat EvenInt OddInt NzInt Int .
subsorts OddNat < OddInt NzNat < NzInt < Int .
subsorts EvenNat < EvenInt Nat < Int .

<!-- manual page 372 -->
360 CHAPTER 13. UNIFICATION
subsorts NzEvenNat < NzNat EvenNat < Nat .
op 0 : -> EvenNat .
op s : EvenNat -> OddNat [iter] .
op s : OddNat -> NzEvenNat [iter] .
op s : Nat -> NzNat [iter] .
op s : EvenInt -> OddInt [iter] .
op s : OddInt -> EvenInt [iter] .
op s : Int -> Int [iter] .
op _+_ : Int Int -> Int [comm gather (E e)] .
op _+_ : OddInt OddInt -> EvenInt [ditto] .
op _+_ : EvenInt EvenInt -> EvenInt [ditto] .
op _+_ : OddInt EvenInt -> OddInt [ditto] .
op _+_ : Nat Nat -> Nat [ditto] .
op _+_ : Nat NzNat -> NzNat [ditto] .
op _+_ : OddNat OddNat -> NzEvenNat [ditto] .
op _+_ : NzEvenNat EvenNat -> NzEvenNat [ditto] .
op _+_ : EvenNat EvenNat -> EvenNat [ditto] .
op _+_ : OddNat EvenNat -> OddNat [ditto] .
endfm
We can then give the unification commands:
Maude> unify in ITER-EXAMPLE :
s^1000000(X:OddNat) =? s^100000000001(Y:Int) .
Unifier 1
X:OddNat --> s^99999000001(#1:EvenNat)
Y:Int --> #1:EvenNat
and
Maude> unify in ITER-EXAMPLE :
s^1000000(X:OddNat) =? s^100000000001(Y:Int + Z:Int + W:Int) .
Unifier 1
X:OddNat --> s^99999000001(#1:OddNat + (#2:OddNat + #3:EvenNat))
W:Int --> #1:OddNat
Z:Int --> #2:OddNat
Y:Int --> #3:EvenNat
Unifier 2
X:OddNat --> s^99999000001(#1:OddNat + (#2:EvenNat + #3:OddNat))
W:Int --> #1:OddNat
Z:Int --> #2:EvenNat
Y:Int --> #3:OddNat
Unifier 3
X:OddNat --> s^99999000001(#1:EvenNat + (#2:OddNat + #3:OddNat))
W:Int --> #1:EvenNat
Z:Int --> #2:OddNat
Y:Int --> #3:OddNat
Unifier 4

<!-- manual page 373 -->
13.4. THE UNIFY COMMAND 361
X:OddNat --> s^99999000001(#1:EvenNat + (#2:EvenNat + #3:EvenNat))
W:Int --> #1:EvenNat
Z:Int --> #2:EvenNat
Y:Int --> #3:EvenNat
Note that the second command produces more unifiers than the first command even though
both unification problems have only one most general unifier. We can, however, obtain just the
single most general unifier for the second unification problem by giving the command:
Maude> irredundant unify in ITER-EXAMPLE :
s^1000000(X:OddNat) =? s^100000000001(Y:Int + Z:Int + W:Int) .
Unifier 1
X:OddNat --> s^99999000001(#1:EvenNat + (#2:EvenNat + #3:EvenNat))
W:Int --> #1:EvenNat
Z:Int --> #2:EvenNat
Y:Int --> #3:EvenNat
As already mentioned, assuming that no bound on the number of unifiers is specified by
the user, Maude will always compute a complete set of order-sorted unifiers modulo Ax, for Ax
the supported equational axioms described in Section 13.3 for which unification is known to be
finitary. However, the unify command does not guarantee that the computed set of unifiers is
minimal, only the irredundant unify command guarantees that. That is, some of the unifiers
in the computed set may be redundant, since they could be obtained as instances (modulo Ax)
of other unifiers in the set.
13.4.4 Associative-commutative with identity (ACU ) unification ex-
amples
To illustrate the use of the unification command in the presence of ACU operators, let us
consider yet another version of the vending machine example (first presented in Section 5.1 and
in other sections of this document in different forms):
mod UNIF-VENDING-MACHINE is
sorts Coin Item Marking Money State .
subsort Coin < Money .
op empty : -> Money .
op __ : Money Money -> Money [assoc comm id: empty] .
subsort Money Item < Marking .
op __ : Marking Marking -> Marking [assoc comm id: empty] .
op <_> : Marking -> State .
ops $ q : -> Coin .
ops a c : -> Item .
var M : Marking .
rl [buy-c] : < M $ > => < M c > .
rl [buy-a] : < M $ > => < M a q > .
eq [change]: q q q q = $ .
endm
We can ask whether there is an equational unifier of two configurations, one containing at
least two quarters, and another containing at least one dollar.
Maude> unify in UNIF-VENDING-MACHINE :
< q q X:Marking > =? < $ Y:Marking > .

<!-- manual page 374 -->
362 CHAPTER 13. UNIFICATION
Unifier 1
X:Marking --> $
Y:Marking --> q q
Unifier 2
X:Marking --> $ #1:Marking
Y:Marking --> q q #1:Marking
Notice that the computed set of unifiers is not minimal, because the first solution is the instance
of the second obtained by substituting the variable #1:Marking with the constant empty.
Maude> irredundant unify in UNIF-VENDING-MACHINE :
< q q X:Marking > =? < $ Y:Marking > .
Unifier 1
X:Marking --> $ #1:Marking
Y:Marking --> q q #1:Marking
Recall that memberships are discarded completely. For instance, if we modify the previous
example to include a membership definition for a new sort Quarter, a unification call with that
sort may not succeed.
mod UNIF-VENDING-MACHINE-MB is
sorts Coin Item Marking Money State .
subsort Coin < Money .
op empty : -> Money .
op __ : Money Money -> Money [assoc comm id: empty] .
subsort Money Item < Marking .
op __ : Marking Marking -> Marking [assoc comm id: empty] .
op <_> : Marking -> State .
ops $ q : -> Coin .
ops a c : -> Item .
sort Quarter .
subsort Quarter < Coin .
mb q : Quarter .
var M : Marking .
rl [buy-c] : < M $ > => < M c > .
rl [buy-a] : < M $ > => < M a q > .
eq [change]: q q q q = $ .
endm
We can ask whether there is an equational unifier of two configurations, one containing at least
two quarters, and another containing two quarters and a dollar, but it fails:
Maude> unify in UNIF-VENDING-MACHINE-MB :
< q q X:Marking > =? < $ Y:Quarter Z:Quarter > .
No unifier.
despite the fact that instantiating both Y and Z to q is part of a solution in the unification call
above. The reason is that the membership is not used during ACU unification and therefore
the algorithm unification treats the sort Quarter as empty.

<!-- manual page 375 -->
13.4. THE UNIFY COMMAND 363
13.4.5 Unification examples with an identity symbol
Let us illustrate the use of the different combinations of the identity attribute for unification.
Let us consider first a module using the left-id attribute.
mod LEFTID-UNIFICATION-EX is
sorts Magma Elem .
subsorts Elem < Magma .
op __ : Magma Magma -> Magma [gather (E e) left id: e] .
ops a b c d e : -> Elem .
endm
Then the following two unification problems have a different meaning, where we have swapped
the position of the variables. First, when we unify two terms where variables of sort Magma are
at the left of the terms, we have both a syntactic unifier and a unifier modulo identity.
Maude> unify in LEFTID-UNIFICATION-EX : X:Magma a =? Y:Magma a a .
Unifier 1
X:Magma --> a
Y:Magma --> e
Unifier 2
X:Magma --> #1:Magma a
Y:Magma --> #1:Magma
Note that the second one is the most general.
Maude> irredundant unify in LEFTID-UNIFICATION-EXAMPLE :
X:Magma a =? Y:Magma a a .
Unifier 1
X:Magma --> #1:Magma a
Y:Magma --> #1:Magma
WhenthevariablesareinsteadattherightsideofthetermsofsortMagma,thereisclearlynouni-
fier,sincetheterma a Y:Magmaisparsedas(a a) Y:MagmainmoduleLEFTID-UNIFICATION-EX
due to the attribute gather (E e) (see Section 3.9).
Maude> unify in LEFTID-UNIFICATION-EX : a X:Magma =? a a Y:Magma .
No unifier.
Consider now a similar module but for the right identity.
mod RIGHTID-UNIFICATION-EX is
sorts Magma Elem .
subsorts Elem < Magma .
op __ : Magma Magma -> Magma [gather (e E) right id: e] .
ops a b c d e : -> Elem .
endm
When we unify two terms where variables of sort Magma are at the left of the terms, there
is clearly no unifier, since the term Y:Magma a a is parsed as Y:Magma (a a) in module
RIGHTID-UNIFICATION-EX due to the attribute gather (e E) (see Section 3.9).
Maude> unify in RIGTHID-UNIFICATION-EX : X:Magma a =? Y:Magma a a .
No unifier.
When the variables are instead at the right side of the terms of sort Magma, we have both a
syntactic unifier and a unifier modulo identity:

<!-- manual page 376 -->
364 CHAPTER 13. UNIFICATION
Maude> unify in RIGHTID-UNIFICATION-EX : a X:Magma =? a a Y:Magma .
Unifier 1
X:Magma --> a
Y:Magma --> e
Unifier 2
X:Magma --> a #1:Magma
Y:Magma --> #1:Magma
Note that the second one is the most general.
Maude> irredundant unify in RIGHTID-UNIFICATION-EX :
a X:Magma =? a a Y:Magma .
Unifier 1
X:Magma --> a #1:Magma
Y:Magma --> #1:Magma
Consider now a similar module but with the identity attribute (no left or right restriction).
mod ID-UNIFICATION-EX is
sorts Magma Elem .
subsorts Elem < Magma .
op __ : Magma Magma -> Magma [gather (E e) id: e] .
ops a b c d e : -> Elem .
endm
When we unify two terms where variables of sort Magma are at the left of the terms, we have
both a syntactic unifier and a unifier modulo identity:
Maude> unify in ID-UNIFICATION-EX : X:Magma a =? Y:Magma a a .
Unifier 1
X:Magma --> a
Y:Magma --> e
Unifier 2
X:Magma --> #1:Magma a
Y:Magma --> #1:Magma
Note that the second one is the most general.
Maude> irredundant unify in ID-UNIFICATION-EX : X:Magma a =? Y:Magma a a .
Unifier 1
X:Magma --> #1:Magma a
Y:Magma --> #1:Magma
When the variables of sort Magma are instead at the right side of the terms of sort Magma, we
only have a unifier modulo identity:
Maude> unify in ID-UNIFICATION-EX : a X:Magma =? a a Y:Magma .
Unifier 1
X:Magma --> a
Y:Magma --> e
And finally, when we add commutativity, we obtain slightly different results.

<!-- manual page 377 -->
13.4. THE UNIFY COMMAND 365
mod COMM-ID-UNIFICATION-EX is
sorts Magma Elem .
subsorts Elem < Magma .
op __ : Magma Magma -> Magma [gather (E e) comm id: e] .
ops a b c d e : -> Elem .
endm
When we unify two terms where variables of sort Magma are at the left of the terms, we have
both a syntactic unifier and a unifier modulo identity and commutativity, but the latter is
duplicated:
Maude> unify in COMM-ID-UNIFICATION-EX: X:Magma a =? Y:Magma a a .
Unifier 1
X:Magma --> a
Y:Magma --> e
Unifier 2
X:Magma --> a #1:Magma
Y:Magma --> #1:Magma
Unifier 3
X:Magma --> a
Y:Magma --> e
Note that the second one is the most general.
Maude> irredundant unify in COMM-ID-UNIFICATION-EX :
X:Magma a =? Y:Magma a a .
Unifier 1
X:Magma --> a #1:Magma
Y:Magma --> #1:Magma
When the variables of sort Magma are instead at the right side of the terms of sort Magma, we
have several unifiers modulo identity and commutativity (indeed, the most general ones):
Maude> unify in COMM-ID-UNIFICATION-EXAMPLE : a X:Magma =? a a Y:Magma .
Unifier 1
X:Magma --> a a
Y:Magma --> a
Unifier 2
X:Magma --> a
Y:Magma --> e
Note that the first solution is intriguing and is obtained by unifying terms (X:Magma a) and
((a a) Y:Magma).
13.4.6 Associative (A) unification examples
In general, unification modulo associativity is not finitary. However, the Maude implementation
provides a finitary and complete set of unifiers for a substantial class of unification problems
where associative unification happens to be finitary. In problems outside of this class, a finite
but in general incomplete set of unifiers is returned, with an explicit warning that the set

<!-- manual page 378 -->
366 CHAPTER 13. UNIFICATION
may be incomplete. Let us illustrate the use of the unification command in the presence of A
symbols with the different capabilities and limitations. One of the key ideas for having finitary
unification modulo associativity is to have variables over associative symbols being linear, i.e.,
having only one occurrence among all the terms in a unification problem.
Consider a very simple module with a binary associative function symbol:
fmod UNIFICATION-EX4 is
protecting NAT .
sort NList .
subsort Nat < NList .
op _:_ : NList NList -> NList [assoc] .
endfm
A unification problem using such an associative function symbol returns five unifiers without
any problem because the unification problem is linear.
Maude> unify in UNIFICATION-EX4 :
X:NList : Y:NList : Z:NList =? P:NList : Q:NList .
Unifier 1
X:NList --> #3:NList : #4:NList
Y:NList --> #1:NList
Z:NList --> #2:NList
P:NList --> #3:NList
Q:NList --> #4:NList : #1:NList : #2:NList
Unifier 2
X:NList --> #1:NList
Y:NList --> #3:NList : #4:NList
Z:NList --> #2:NList
P:NList --> #1:NList : #3:NList
Q:NList --> #4:NList : #2:NList
Unifier 3
X:NList --> #1:NList
Y:NList --> #2:NList
Z:NList --> #4:NList : #3:NList
P:NList --> #1:NList : #2:NList : #4:NList
Q:NList --> #3:NList
Unifier 4
X:NList --> #1:NList
Y:NList --> #2:NList
Z:NList --> #3:NList
P:NList --> #1:NList : #2:NList
Q:NList --> #3:NList
Unifier 5
X:NList --> #1:NList
Y:NList --> #2:NList
Z:NList --> #3:NList
P:NList --> #1:NList
Q:NList --> #2:NList : #3:NList
The unification problem may not be linear but it may be easy to detect that there is

<!-- manual page 379 -->
13.4. THE UNIFY COMMAND 367
no unifier, e.g. it is impossible to unify a list X concatenated with itself with another list Y
concatenated also with itself but with a natural number, e.g. 1, in between.
Maude> unify in UNIFICATION-EX4 :
X:NList : X:NList =? Y:NList : 1 : Y:NList .
No unifier.
When associative variables are non-linear, Maude has implemented a cycle detection that
may have different outcomes:
1. There are non-linear variables but only on one side of the problem. This implies that
there is no risk of infinitary behavior and Maude will be complete with no need for cycle
detection.
Maude> unify in UNIFICATION-EX4 :
P:NList : P:NList =? 1 : Q:NList : 2 .
Unifier 1
P:NList --> 1 : #1:NList : 2
Q:NList --> #1:NList : 2 : 1 : #1:NList
Unifier 2
P:NList --> 1 : 2
Q:NList --> 2 : 1
2. Cycle detection is used when non-linear variables appear on both sides but no variable
occurs more than twice. It may detect spurious cycles, i.e., a cycle that does not corre-
spond to unifiers, but then nothing is reported. If there is at least one unifier associated
to the cycle, a warning will be printed and the acyclic solutions are returned.
Maude> unify in UNIFICATION-EX4 : 0 : Q:NList =? Q:NList : 0 .
Warning: Unification modulo the theory of operator _:_ has
encountered an instance for which it may not be complete.
Unifier 1
Q:NList --> 0
Warning: Some unifiers may have been missed due to incomplete
unification algorithm(s).
Note that the unification problem 0:X=? X:0 has an infinite family of most general
unifiers {X →0n}for 0n being a list of n consecutive 0 elements.
3. If neither of the two above cases apply, then Maude forces termination with an internal
depth bound and, therefore, no cycle detection is activated. In this case, a warning will
be printed and the solutions up to the internal depth bound are returned.
Maude> unify in UNIFICATION-EX4 :
X:NList : X:NList : X:NList
=? Y:NList : Y:NList : Z:NList : Y:NList .
Warning: Unification modulo the theory of operator _:_ has
encountered an instance for which it may not be complete.
Unifier 1
X:NList --> #1:NList : #1:NList : #1:NList : #1:NList
Y:NList --> #1:NList : #1:NList : #1:NList
Z:NList --> #1:NList : #1:NList : #1:NList
Unifier 2

<!-- manual page 380 -->
368 CHAPTER 13. UNIFICATION
X:NList --> #1:NList : #1:NList : #1:NList
Y:NList --> #1:NList : #1:NList
Z:NList --> #1:NList : #1:NList : #1:NList
Unifier 3
X:NList --> #1:NList : #1:NList
Y:NList --> #1:NList
Z:NList --> #1:NList : #1:NList : #1:NList
Warning: Some unifiers may have been missed due to incomplete
unification algorithm(s).
If the verbose mode is activated (see Appendix A.15), Maude will print different internal
messages associated to the situations above:
• Associative unification using cycle detection.
• Associative unification algorithm detected an infinite family of unifiers.
• Associative unification using depth bound of 7.
• Associative unification algorithm hit depth bound.
Note that the depth bound can be changed from the command line, see Appendix A.1.
13.4.7 Associative with identity (AU ) unification examples
Unification modulo associativity and identity was the missing case for associativity without
commutativity until Maude 3.1. Unification modulo associativity but with left identity or right
identityisstillmissingandwillbeavailableinthefuture. However, itiscurrentlysupportedina
different way, namely, by variant-based unification using an explicit equation (see Section 14.8).
Note that unification modulo associativity and identity is, in general, not finitary, as in the case
of only associativity, and the same conventions and warnings apply (see Section 13.4.6). Let
us illustrate the use of the unification command in the presence of AU symbols using previous
examples.
Consider a very simple module, adapting the previous module UNIFICATION-EX4, with a
binary associative symbol with an identity symbol for the empty list:
fmod UNIFICATION-EX5 is
protecting NAT .
sort NList .
op nil : -> NList .
subsort Nat < NList .
op _:_ : NList NList -> NList [assoc id: nil] .
endfm
The unification problem below, using an associative symbol with identity, returns 32 unifiers
using the unify command. Instead, the minimal set contains only 3, obtained by using the
irredundant unify command.
Maude> irredundant unify in UNIFICATION-EX5 :
X:NList : Y:NList : Z:NList =? P:NList : Q:NList .
Unifier 1
X:NList --> #3:NList : #4:NList
Y:NList --> #1:NList
Z:NList --> #2:NList
P:NList --> #3:NList
Q:NList --> #4:NList : #1:NList : #2:NList

<!-- manual page 381 -->
13.4. THE UNIFY COMMAND 369
Unifier 2
X:NList --> #1:NList
Y:NList --> #3:NList : #4:NList
Z:NList --> #2:NList
P:NList --> #1:NList : #3:NList
Q:NList --> #4:NList : #2:NList
Unifier 3
X:NList --> #1:NList
Y:NList --> #2:NList
Z:NList --> #4:NList : #3:NList
P:NList --> #1:NList : #2:NList : #4:NList
Q:NList --> #3:NList
This is an example where associativity-identity has fewer most general unifiers than associativ-
ity; see the unification command for theory UNIFICATION-EX4 in page 366 above with 5 most
general unifiers (both the unify and irredundant unify commands return 5 although only
the former is shown).
The reader may be aware that the algorithm for associative-identity unification produces
many redundant unifiers due to its collapse nature and how the order-sorted information is
actually treated. Therefore, the use of the irredundant unify command is advised in this
case.

<!-- manual page 382 -->
370 CHAPTER 13. UNIFICATION

