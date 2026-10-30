# Chapter 14: Variants and Variant Unification (manual pages 383-398)

<!-- manual page 383 -->
Chapter 14
Variants and Variant Unification
14.1 Introduction
As explained in Section 13.2, Maude features order-sorted unification modulo axioms Ax,
including associativity1 (A), commutativity (C), associative-identity1 (AU), commutativity-
identity(CU),associativity-commutativity(AC),associativity-commutativity-identity(ACU),
and (left-, right- or two-sided) identity (Ul, Ur, U). However, order-sorted equational unifica-
tion in full generality considers a decomposition of an equational theory (Σ,E∪Ax) into two
problems: one of Ax-unification, and another of E∪Ax-unification that uses an Ax-unification
algorithm as a subroutine. As explained in Sections 13.2.1, algorithms for E∪Ax-unification
have been extensively defined by using narrowing-based unification, where for a unification
problem t=? t′ we obtain the search space associated to narrowing the term eq(t,t′) using E
modulo Ax and search for all paths from eq(t,t′) to the truth constant tt. However, we use
the notion of variants of a term for generating such a narrowing search space.
14.2 Term variants
Comon-Lundh and Delaune’s notion of variant [30] characterizes the instances of a term w.r.t.
an equational theory E∪Axsuch that the equations Eare confluent, terminating, and coherent
modulo axioms Ax.
The E,Ax-variants of a term t are pairs (t′,θ), with θ a substitution and t′ the
E,Ax-canonical form of θ(t).
A preorder relation of generalization that holds between such pairs can be given: we say a
variant (t1,θ1) of a term t is more general than another variant (t2,θ2) of the same term t if
there is a substitution ρ such that ρ(t1) =Ax t2, and (θ1; ρ)|Var(t)=Ax (θ2)|Var(t). A complete
set of E,Ax-variants (up to renaming) of a term t is a subset V of E,Ax-variants of t such
that for any variant (σ(t)↓E,Ax,σ) there exists a more general variant (t′,θ) in V.
In order to avoid clashing of algorithms and notions, we have decided that the equations
used for variant generation (and variant-based unification) should be identifiable and clearly
distinguished from standard equations in Maude. For this purpose we have defined a new
1As already explained, for a useful subclass of associative and associative-identity unification problems com-
plete sets of unifiers are returned, and in all other cases a possibly incomplete such set is returned with a
warning. See Sections 13.4.6 and 13.4.7.
371

<!-- manual page 384 -->
372 CHAPTER 14. VARIANTS AND VARIANT UNIFICATION
attribute for equations: the keyword variant. This implies that if the user wants to use an
equation t = t’ both for variant generation and for simplification, it should be duplicated:
eq t = t’ . and eq t = t’ [variant] . No equation with the variant attribute can have
the owise attribute. Note that this allows a greater flexibility at the operational level when
combining variant generation and simplification: by the above method, an equation can be
used for either purpose (declared only once in the appropriate way), or for both, by a double
declaration.
For example, consider the following functional module defining the addition function _+_
on natural numbers built from 0 and s:
fmod NAT-VARIANT is
sort Nat .
op 0 : -> Nat [ctor] .
op s : Nat -> Nat [ctor] .
op _+_ : Nat Nat -> Nat .
vars X Y : Nat .
eq [base] : 0 + Y = Y [variant] .
eq [ind] : s(X) + Y = s(X + Y) [variant] .
endfm
The term X + s(0) has an infinite number of variants w.r.t. those equations, i.e.,
• (s(0),{X →0}),
• (s(Y + s(0)),{X →s(Y)}),
• (s(s(0)),{X →s(0)}),
• (s(s(Y + s(0))),{X →s(s(Y))}),
• (s(s(s(0))),{X →s(s(0))}),...
Indeed, there is no finite, complete, most general set of variants for that term. However, the
term 0 + X has a finite number of most general variants w.r.t. those equations, i.e., (X,id).
Obviously, there are many more variants, such as (0,{X →0}), but they are all instances of
the most general one.
An equational theory E∪Axhas the finite variant property (or is FVP) iff there is a finite
complete set of most general variants for each term. This property also ensures the existence
of a generic finitary E∪Ax-unification algorithm based on computing variants, as shown in
Section 14.8. However, if a theory does not have the finite variant property, we are still able to
incrementally enumerate all the variants of a term, as explained below in Section 14.6.
Consider the following equational theory for exclusive or.
fmod EXCLUSIVE-OR is
sorts Nat NatSet .
op 0 : -> Nat [ctor] .
op s : Nat -> Nat [ctor] .
subsort Nat < NatSet .
op mt : -> NatSet [ctor] .
op _*_ : NatSet NatSet -> NatSet [ctor assoc comm] .
vars X Y Z W : [NatSet] .
eq [idem] : X * X = mt [variant] .
eq [idem-Coh] : X * X * Z = Z [variant] .
eq [id] : X * mt = X [variant] .
endfm
This theory has the finite variant property, as proved manually in [30] and automatically in [61].
For instance, the term X * X has a finite set of most general variants, just (mt,id). And the

<!-- manual page 385 -->
14.3. THEORIES CURRENTLY SUPPORTED 373
term X * Y has also a finite, complete set of most general variants:
1. (X * Y,id),
2. (Z,{X →mt,Y →Z}),
3. (Z,{X →Z,Y →mt}),
4. (Z,{X →Z * U,Y →U}),
5. (Z,{X →U,Y →Z * U}),
6. (mt,{X →U,Y →U}), and
7. (Z1 * Z2,{X →U * Z1,Y →U * Z2})
Note that if variable X in the equational theory is changed2 from sort [NatSet] to Nat,
then the theory does not have the finite variant property, since every pair of similar elements
has to be separately eliminated, whereas now chunks of similar elements can be eliminated at
once. Also, note that the symbol _*_ cannot be made ACU instead of AC, because then the
equation3 X * X * Z = Z is not ACU-terminating, since for any term t, we have Z →t and
X →mt, and then t rewrites to itself.
The finite variant property happens to be an undecidable problem [13]. However, a semi-
decision procedure for checking the finite variant property has been developed which works well
in practice: it has been shown [18] that, in order to prove the finite variant property for an
equational theory (Σ,E∪Ax), it is enough to check, for each function symbol f ∈Σ, whether
or not each pattern of the form f(X1,...,Xn) has a finite number of variants, where the Xi
are distinct variables of the appropriate kind and n is the arity of f. This can be done by
attempting to generate all the variants of f(X1,...,Xn) as described in Section 14.4 below.
In Section 14.3 we introduce the class of equational theories admissible for variant genera-
tion, and thus for variant-based unification. We also provide in Section 14.4 two commands for
user generation of variants. Variants are used for variant-based unification in Sections 14.8 and
14.9, and such a variant-based unification is later used in Section 15.6 for symbolic reachability
analysis.
14.3 Theories currently supported
The equational theories that are admissible for variant generation are as follows. Let fmod
(Σ,E ∪Ax) endfm (resp. fth (Σ,E ∪Ax,R) endfth) be an order-sorted functional module
(resp. functional theory) where E is a set of equations specified with the eq keyword and the
attributevariant,andAxisasetofaxiomssuchthat(Σ,Ax) satisfiestherestrictionsexplained
in Section 13.3. Furthermore, the equations E must satisfy the following extra conditions:
• The equations E are unconditional, confluent, terminating, sort-decreasing, and coherent
modulo Ax (see [96, 97] for coherence details).
• An equation’s lefthand side cannot be a variable, and the owise feature is not allowed.
• Equations having both the variant and nonexec attributes produce an advisory, and are
ignored for variant generation.
Any system module mod (Σ,G∪E∪Ax,R) endm (or system theory th (Σ,G∪E∪Ax,R)
endth), where Gis an additional set of equations (without the variant attribute!) and R is a
setofrules, isalsoconsideredadmissibleforvariantgenerationiftheequationalpart(Σ,E∪Ax)
satisfies the conditions described above. Note that when an equational theory (Σ,G∪E∪Ax)
is entered into Maude, each equation in E (used for variant computation) must include the
variant attribute. Note that equations in G do not have any restriction, i.e., they can be
conditional equations, with the owise attribute, etc.
2Nothing happens if X is changed from the kind [NatSet] to the kind [Nat], since NatSet and Nat are in the
same kind.
3This is the only equation necessary for ACU -coherence and the other two would be eliminated.

<!-- manual page 386 -->
374 CHAPTER 14. VARIANTS AND VARIANT UNIFICATION
14.4 The get variants command
Given a module ⟨ModId⟩, Maude provides two variant generation commands of the form:
get variants [ n ] in ⟨ModId⟩ : ⟨Term⟩.
get irredundant variants [ n ] in ⟨ModId⟩ : ⟨Term⟩.
where n is an optional argument providing a bound on the number of variants requested, so
that if the cardinality of the set of variants is greater than the specified bound, the variants
beyond that bound are omitted; and, as usual, if no module is mentioned, the current module
is used.
Maude allows an incremental generation of variants, as described in Section 14.6 below.
When a theory does not have the finite variant property, an incremental generation of the
(possibly infinite) set of most general variants would be returned by the first command get
variants. However, the second command, get irredundant variants, is useful for theories
that do have the finite variant property, since it will provide the set of most general variants of
a term, which is the basis for variant-based unification in Section 14.8.
For example, we can check that the EXCLUSIVE-OR module above has the finite variant
property by simply generating the variants for the exclusive-or symbol ∗.
Maude> get irredundant variants in EXCLUSIVE-OR : X * Y .
Variant 1
[NatSet]: #1:[NatSet] * #2:[NatSet]
X --> #1:[NatSet]
Y --> #2:[NatSet]
Variant 2
NatSet: mt
X --> %1:[NatSet]
Y --> %1:[NatSet]
Variant 3
[NatSet]: %1:[NatSet] * %3:[NatSet]
X --> %1:[NatSet] * %2:[NatSet]
Y --> %2:[NatSet] * %3:[NatSet]
Variant 4
[NatSet]: %1:[NatSet]
X --> %1:[NatSet] * %2:[NatSet]
Y --> %2:[NatSet]
Variant 5
[NatSet]: %2:[NatSet]
X --> %1:[NatSet]
Y --> %1:[NatSet] * %2:[NatSet]
Variant 6
[NatSet]: %1:[NatSet]
X --> mt
Y --> %1:[NatSet]
Variant 7
[NatSet]: %1:[NatSet]

<!-- manual page 387 -->
14.4. THE GET VARIANTS COMMAND 375
X --> %1:[NatSet]
Y --> mt
No more variants.
The above example illustrates a difference between unifiers returned by the built-in unifi-
cation modulo axioms and unifiers returned by variant generation or variant-based unification:
there are two forms of fresh variables, the former #n:Sort and the new %n:Sort. The reasons
for this distinction are immaterial: they are adopted conventions dictated by implementation
choices. Both forms represent fresh variables and both share the same counter for new fresh
variables. The user is required not to use variables of any of these two forms in submitted uni-
fication problems (either modulo axioms or variant-based). When variant-based unification is
used at the metalevel (see Section 17.6.9), there are two possibilities: (i) a counter for new fresh
variables must take into account the numbers used in the variables of the unification problem,
or (ii) an identifier among ’#, ’%, and ’@ used in the variables of the unification problem.
Noticethatmembershipsarediscardedcompletely. Forinstance, wecanmodifytheprevious
example to include a membership definition for a new sort Empty.
fmod EXCLUSIVE-OR-MB is
sorts Nat NatSet .
op 0 : -> Nat [ctor] .
op s : Nat -> Nat [ctor] .
subsort Nat < NatSet .
op mt : -> NatSet [ctor] .
op _*_ : NatSet NatSet -> NatSet [ctor assoc comm] .
sort Empty .
subsort Empty < NatSet .
mb mt : Empty .
vars X Y Z : [NatSet] .
eq [idem] : X * X = mt [variant] .
eq [idem-Coh] : X * X * Z = Z [variant] .
eq [id] : X * mt = X [variant] .
endfm
We can ask then for the variants of the exclusive-or symbol ∗when restricted to the sort Empty,
and the results use the variant equations but not the membership information.
Maude> get irredundant variants in EXCLUSIVE-OR-MB : X:Empty * Y:Empty .
Variant 1
NatSet: #1:Empty * #2:Empty
X:Empty --> #1:Empty
Y:Empty --> #2:Empty
Variant 2
Empty: mt
X:Empty --> %1:Empty
Y:Empty --> %1:Empty
No more variants.

<!-- manual page 388 -->
376 CHAPTER 14. VARIANTS AND VARIANT UNIFICATION
Note that the membership is used to compute the least sort of terms involved in the results (like
the constant mt above), but is not used during variant generation. For example, this process is
not able to instantiate any of the two variables to the constant mt.
Consider now the following version of the vending machine to buy apples (a) or cakes (c)
with dollars ($) and/or quarters (q). The reader can check that the only difference with the
UNIF-VENDING-MACHINE module in Section 13.4.4 is the change equation, where we have added
theattributevariantandavariableMtomakeitACU-coherent(see[96,97]forACU-coherence
details).
mod VARIANT-VENDING-MACHINE is
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
eq [change] : q q q q M = $ M [variant] .
endm
Note that the change equation satisfies the finite variant property, as proved by generating the
variants of symbol__
.
Maude> get irredundant variants in VARIANT-VENDING-MACHINE :
X:Marking Y:Marking .
Variant 1
Marking: #1:Marking #2:Marking
X:Marking --> #1:Marking
Y:Marking --> #2:Marking
Variant 2
Marking: $ %1:Marking %2:Marking
X:Marking --> q q q %1:Marking
Y:Marking --> q %2:Marking
Variant 3
Marking: $ %1:Marking %2:Marking
X:Marking --> q q %1:Marking
Y:Marking --> q q %2:Marking
Variant 4
Marking: $ %1:Marking %2:Marking
X:Marking --> q %1:Marking
Y:Marking --> q q q %2:Marking
We can also generate the variants of the state < $ q q X:Marking > containing at least a
dollar and two quarters.
Maude> get irredundant variants in VARIANT-VENDING-MACHINE :
< $ q q X:Marking > .

<!-- manual page 389 -->
14.4. THE GET VARIANTS COMMAND 377
Variant 1
State: < $ q q #1:Marking >
X:Marking --> #1:Marking
Variant 2
State: < $ $ %1:Marking >
X:Marking --> q q %1:Marking
These two variants form a finite, complete, and most general set of variants for the given term;
for example, the variant
(< $ $ q q Y:Marking >, X:Marking --> q q q q Y:Marking)
is an instance of the first variant above, i.e., the canonical form < $ $ q q Y:Marking > is an
instance of the normal form < $ q q #1:Marking > of the first variant, and the (normalized
version) of the instantiating substitution, i.e., #1:Marking --> $ Y:Marking, is an instance of
the empty substitution of the first variant.
We can consider a more complex equational theory such as Lankford’s formalization of an
Abelian group, specified in the following module.
fmod ABELIAN-GROUP is
sorts Element .
op _+_ : Element Element -> Element [comm assoc prec 30] .
op -_ : Element -> Element [prec 20] .
op 0 : -> Element .
vars X Y Z : Element .
eq X + 0 = X [variant] .
eq X + - X = 0 [variant] .
eq X + - X + Y = Y [variant] .
eq - - X = X [variant] .
eq - 0 = 0 [variant] .
eq - X + - Y = -(X + Y) [variant] .
eq -(X + Y) + Y = - X [variant] .
eq -(- X + Y) = X + - Y [variant] .
eq - X + - Y + Z = -(X + Y) + Z [variant] .
eq -(X + Y) + Y + Z = - X + Z [variant] .
endfm
Thegenerationofthevariantsfortheadditionsymboltakesmoretimeandprovides47 variants:
Maude> get irredundant variants in ABELIAN-GROUP : X + Y .
Variant 1
Element: #1:Element + #2:Element
X --> #1:Element
Y --> #2:Element
Variant 2
Element: %1:Element
X --> 0
Y --> %1:Element
...
Variant 46
Element: %2:Element + - (%3:Element + %4:Element)

<!-- manual page 390 -->
378 CHAPTER 14. VARIANTS AND VARIANT UNIFICATION
X --> %5:Element + - (%1:Element + %3:Element)
Y --> %1:Element + %2:Element + - (%4:Element + %5:Element)
Variant 47
Element: - (%2:Element + %3:Element)
X --> %4:Element + - (%1:Element + %2:Element)
Y --> %1:Element + - (%3:Element + %4:Element)
And the minus sign symbol has four variants:
Maude> get irredundant variants in ABELIAN-GROUP : - X .
Variant 1
Element: - #1:Element
X --> #1:Element
Variant 2
Element: %1:Element
X --> - %1:Element
Variant 3
Element: 0
X --> 0
Variant 4
Element: %1:Element + - %2:Element
X --> %2:Element + - %1:Element
14.5 Variant generation with irreducibility constraints
Variant generation is useful for many applications (see Section 15.2) but especially for variant-
based unification, as explained in Section 14.8 below. One special version of the variant genera-
tion algorithm included in Maude 3 includes irreducibility conditions, i.e., a term is not allowed
to be reducible after applying a substitution. This is useful when, for example, the variants of
a term may have been generated before and one is not interested in generating the variants of a
variant of a term, since the variant t3 of a variant t2 of a term t1 must be already covered by a
variant of t1 (maybe t2 itself). The irreducibility condition is appended by using the keywords
such that T1, T2, T3, ... irreducible.
For example, we may reconsider the generation of the set of irredundant variants of the
expression X * Y in page 374 but now assuming that the term itself is irreducible, that is, no
further variant must be computed.
Maude> get irredundant variants in EXCLUSIVE-OR :
X * Y such that X * Y irreducible .
Variant 1
[NatSet]: #1:[NatSet] * #2:[NatSet]
X --> #1:[NatSet]
Y --> #2:[NatSet]
Similarly, we may reconsider the generation of the set of irredundant variants of the state
< $ q q X:Marking > in page 376 but assuming that different subterms separated by commas
are irreducible.

<!-- manual page 391 -->
14.6. INCREMENTAL VARIANT GENERATION 379
Maude> get irredundant variants in VARIANT-VENDING-MACHINE :
< $ q q X:Marking >
such that q q X:Marking, q X:Marking, X:Marking irreducible .
Variant 1
State: < $ q q #1:Marking >
X:Marking --> #1:Marking
This command clearly discards the variant where X:Marking is mapped to q q %1:Marking,
since it violates the condition that q q X:Marking must be irreducible under substitution.
14.6 Incremental variant generation
Another interesting feature is that variant generation is incremental and in this way we are able
to give partial support to theories that do not have the finite variant property. Let us consider
the functional module for addition NAT-VARIANT in Section 14.1 that does not have the finite
variant property. On the one hand, it is possible to have a term with a finite number of most
general variants although the theory does not have the finite variant property. For instance,
the term s(0) + X is simplified into s(X).
Maude> get variants in NAT-VARIANT : s(0) + X:Nat .
Variant 1
Nat: s(#1:Nat)
X:Nat --> #1:Nat
On the other hand, we can approximate the number of variants of a term that we suspect does
not have a finite number of most general variants. For instance, the term X + s(0) has an
infinite number of most general variants and we can approximate that infinite set of variants
by including a bound in the command, as it is also done for unification modulo axioms (see
Section 13.4.2).
Maude> get variants [10] in NAT-VARIANT : X:Nat + s(0) .
Variant 1
Nat: #1:Nat + s(0)
X:Nat --> #1:Nat
Variant 2
Nat: s(0)
X:Nat --> 0
...
Variant 10
Nat: s(s(s(s(s(0)))))
X:Nat --> s(s(s(s(0))))
Note that typing get irredundant variants in the previous command would force Maude to
generate the infinite set of variants before collecting the ten most general ones.
This incremental variant generation may be useful in applications where obtaining a com-
plete set of variants when possible is important. If we do not know a priori whether a term
has a finite number of most general variants, we can incrementally increase the bound and if

<!-- manual page 392 -->
380 CHAPTER 14. VARIANTS AND VARIANT UNIFICATION
we obtain a number of variants smaller than the bound, we know for sure that it had a finite
number of most general variants.
When an equational theory satisfies the requirements in Section 14.3 and does not have the
finite variant property, it is because there are terms with an infinite set of most general variants.
Of course, if the user does not provide a bound, Maude does not stop because it always returns
a complete set of variants. However, when the user provides such a bound, the process will
always terminate with Maude returning a finite set of variants. As said above, the number of
returned variants can be smaller than the given bound when the term has indeed a finite set of
variants, but it will coincide with the bound otherwise.
14.7 Variantgenerationinincompleteunificationexamples
The unification infrastructure supports the notion of incomplete unification algorithms (see
Section 13.4.6) and variant generation and variant unification take it into consideration.
Let us consider an equational theory combining variant equations and associativity.
fmod VARIANT-UNIFICATION-ASSOC is
protecting NAT .
sort NList .
subsort Nat < NList .
op _:_ : NList NList -> NList [assoc ctor] .
var E : Nat .
var L : NList .
ops tail prefix : NList ~> NList .
ops head last : NList ~> Nat .
eq head(E : L) = E [variant] .
eq tail(E : L) = L [variant] .
eq prefix(L : E) = L [variant] .
eq last(L : E) = E [variant] .
op duplicate : NList ~> Bool .
eq duplicate(L : L) = true [variant] .
endfm
Some terms have a finite set of most general variants modulo associativity.
Maude> get variants in VARIANT-UNIFICATION-ASSOC : head(prefix(tail(L))) .
Variant 1
Nat: head(prefix(tail(#1:NList)))
L --> #1:NList
Variant 2
Nat: head(prefix(%2:NList))
L --> %1:Nat : %2:NList
Variant 3
Nat: head(#2:NList)
L --> #1:Nat : #2:NList : #3:Nat

<!-- manual page 393 -->
14.7. VARIANT GENERATION IN INCOMPLETE UNIFICATION EXAMPLES 381
Variant 4
Nat: %3:Nat
L --> %1:Nat : %3:Nat : %4:NList : %2:Nat
No more variants.
However, some terms may hit incomplete associative unification calls (see Section 13.4.6), and
an incompleteness warning for associative unification will be printed.
Maude> get variants in VARIANT-UNIFICATION-ASSOC :
duplicate(prefix(L) : tail(L)) .
Variant 1
[Bool]: duplicate(prefix(#1:NList) : tail(#1:NList))
L --> #1:NList
Variant 2
[Bool]: duplicate(%1:NList : tail(%1:NList : %2:Nat))
L --> %1:NList : %2:Nat
Variant 3
[Bool]: duplicate(prefix(%1:Nat : %2:NList) : %2:NList)
L --> %1:Nat : %2:NList
Variant 4
[Bool]: duplicate(#1:Nat : #2:NList : #2:NList : #3:Nat)
L --> #1:Nat : #2:NList : #3:Nat
Variant 5
[Bool]: duplicate(#1:Nat : #2:Nat)
L --> #1:Nat : #2:Nat
Warning: Unification modulo the theory of operator _:_ has encountered
an instance for which it may not be complete.
Variant 6
Bool: true
L --> %1:Nat : %1:Nat : %1:Nat
Variant 7
Bool: true
L --> %1:Nat : %1:Nat
No more variants.
Warning: Some variants may have been missed due to incomplete
unification algorithm(s).
Note that the term duplicate(prefix(L) : tail(L)) has an infinite set of most general vari-
antsforthecaseofreturningthevarianttermtrue, i.e., thefamilyofsubstitutions{L:NList →
N:Nat :···: N:Nat}. This is due to the associative unification call (N:Nat:L:NList) =?
(L:NList:N:Nat) invoked internally by the variant generation process (see Section 13.4.6 for
a similar associative unification problem with an infinite set of most general unifiers).

<!-- manual page 394 -->
382 CHAPTER 14. VARIANTS AND VARIANT UNIFICATION
14.8 Variant-based equational order-sorted unification
The intimate connection between E,Ax-variants and E∪Ax-unification is as follows. Suppose
that we extend the equational theory (Σ,E∪Ax) to (Σ,E∪Ax) by adding to Σ a new sort
Truth, not related to any sort in Σ, with a constant tt, and, for each top sort of a connected
component [s], an operator eq : [s] [s] -> Truth; and where E is the result of adding for
each top sort [s] an extra (oriented) equation eq(x,x) = tt (where x is a variable of sort
[s]) to E. Then, given any two terms t,t′, if θ is an E ∪Ax-unifier of t and t′, then the
E,Ax-canonical forms of θ(t) and θ(t′) must be Ax-equal and therefore the pair (tt,θ) must be
a variant of the term eq(t,t′). Furthermore, if the term eq(t,t′) has a finite set of most general
variants, then we are guaranteed that the set of most general E∪Ax-unifiers of tand t′is finite.
At a practical level, variants are generated using narrowing (see Chapter 15 for narrowing
capabilities in Maude). Narrowing with oriented equations E (with or without modulo Ax)
enjoys well-known completeness results, including the generation of complete sets of unifiers
and covering all rewriting sequences from instances of a term using normalized substitutions
(i.e., variants). For instance, [74] showed that narrowing with E without axioms enjoyed
good completeness results, and [75] showed that narrowing with E modulo axioms Axenjoyed
also good completeness results. But narrowing can be quite inefficient, generating a huge
search space, and different narrowing strategies have been devised to reduce the search space
while remaining complete, specially for unification purposes (see [4] for a survey on narrowing
termination). The basic narrowing strategy of [74] provided a restriction of narrowing that,
while being complete, it was terminating for specific classes of theories. However, very little
was known about effective narrowing strategies in the modulo case, and some of the known
anomalies ring a cautionary note, to the effect that the naive extensions of standard narrowing
strategies, for example basic narrowing modulo AC, were incomplete [115, 30], although a
variation of the basic narrowing strategy has been proved complete in [78]. In [62], the folding
variant narrowing strategy is defined for the modulo case and it is proved to be complete for
variants and with good termination properties, providing a finitary and complete unification
algorithm for equational unification for the theories described in Section 14.3 that also satisfy
the finite variant property. Moreover, it is even better than the basic narrowing strategy in
the case without axioms, since it can terminate for equational theories where basic narrowing
cannot (see [62]).
14.9 The variant unify command
Given a module ⟨ModId⟩, of the general form mod (Σ,G∪E∪Ax,R) endm where (Σ,E∪Ax)
satisfies the requirements of Section 14.3 and satisfies also the finite variant property, Maude
provides a command for E ∪Ax-equational unification based on variant generation of the
following two forms:
variant unify [ n ] in ⟨ModId⟩ :
⟨Term-1⟩ =? ⟨Term’-1⟩ /\ ... /\ ⟨Term-k⟩ =? ⟨Term’-k⟩.
filtered variant unify [ n ] in ⟨ModId⟩ :
⟨Term-1⟩ =? ⟨Term’-1⟩ /\ ... /\ ⟨Term-k⟩ =? ⟨Term’-k⟩.
where k≥1; nis an optional argument providing a bound on the number of unifiers requested,
so that if the cardinality of the set of unifiers is greater than the specified bound, the unifiers
beyond that bound are omitted; and, as usual, if no module is mentioned, the current module
is used. The second command generates all the unifiers and, then, filters them against each
other in order to return a minimal set of most general unifiers modulo the equational theory.

<!-- manual page 395 -->
14.10. VARIANT-BASED UNIFICATION WITH IRREDUCIBILITY CONSTRAINTS 383
Consider again the module VARIANT-VENDING-MACHINE introduced in Section 14.4. We can
ask whether there is an E∪Ax-equational unifier of two configurations, one containing at least
two quarters, and another containing at least one dollar, by invoking the following command:
Maude> variant unify in VARIANT-VENDING-MACHINE :
< q q X:Marking > =? < $ Y:Marking > .
Unifier 1
X:Marking --> $ %1:Marking
Y:Marking --> q q %1:Marking
Unifier 2
X:Marking --> q q #1:Marking
Y:Marking --> #1:Marking
It may not be obvious that this is not the minimal set of most general unifiers, but the filtered
version returns only one unifier.
Maude> filtered variant unify in VARIANT-VENDING-MACHINE :
< q q X:Marking > =? < $ Y:Marking > .
Unifier 1
X:Marking --> q q #1:Marking
Y:Marking --> #1:Marking
Thefirstunifierisaninstanceofthesecondunifierbyapplying#1:Marking →q q %1:Marking
and simplifying q q q q into $. Instead, the second unifier is not an instance of the first unifier
because the $ in normal form disables any possibility.
Note that there are equational theories where two unifiers are comparable in both directions,
i.e. unifier σ1 is an instance of unifier σ2 modulo the equational theory and viceversa. In such
a case, Maude arbitrarily returns one of them. For instance, it is well known that unification in
the exclusive-or theory is unitary [76] and, for the following unification problem, the variant
unify command returns 57 unifiers whereas the filtered variant unify command returns
just one, but several unifiers could be appropriate candidates for most general unifier.
Maude> filtered variant unify in EXCLUSIVE-OR : X * Y =? Z * W .
Unifier 1
X --> %1:[NatSet] * %3:[NatSet]
Y --> %2:[NatSet] * %4:[NatSet]
Z --> %1:[NatSet] * %2:[NatSet]
W --> %3:[NatSet] * %4:[NatSet]
14.10 Variant-basedunificationwithirreducibilityconstraints
Similarly to Section 14.5, we may need to perform variant-based unification but with some
irreducibility conditions. The irreducibility condition is appended by using the keywords
such that T1, T2, T3, ... irreducible.
For example, we may reconsider the unification problem of terms < q q X:Marking > and
< $ Y:Marking > above but assuming that different subterms separated by commas are irre-
ducible.
Maude> variant unify in VARIANT-VENDING-MACHINE :
< q q X:Marking > =? < $ Y:Marking >

<!-- manual page 396 -->
384 CHAPTER 14. VARIANTS AND VARIANT UNIFICATION
such that q q X:Marking, q X:Marking, X:Marking irreducible .
Unifier 1
rewrites: 0 in 0ms cpu (0ms real) (0 rewrites/second)
X:Marking --> $ %1:Marking
Y:Marking --> q q %1:Marking
ThiscommandclearlydiscardsthesecondunifierwhereX:Markingismappedtoq q %1:Marking,
since it violates the condition that q q X:Marking must be irreducible under substitution.
14.11 Incremental variant unification
Similarly to the incremental generation of variants in Section 14.6, one can obtain an incremen-
tal number of unifiers for a given unification problem. Let us consider again the NAT-VARIANT
module in Section 14.1 that does not have the finite variant property. On the one hand, it is
possible to have a finite number of most general unifiers for a unification problem although the
theory does not have the finite variant property. For instance, the unification problem between
s(0) + X and s(s(s(0))) returns just one unifier.
Maude> variant unify in NAT-VARIANT : s(0) + X:Nat =? s(s(s(0))) .
Unifier 1
X:Nat --> s(s(0))
No more unifiers.
On the other hand, we can approximate the number of unifiers of a unification problem that
we suspect does not have a finite number of most general unifiers. For instance, the unification
problem between terms X + s(0) and s(s(s(0))) has only one solution X →s(s(0)) and we
can obtain that solution by including a bound in the command, as it is also done for variant
generation.
Maude> variant unify [1] in NAT-VARIANT : X:Nat + s(0) =? s(s(s(0))) .
Unifier 1
X:Nat --> s(s(0))
However, if we tried to obtain two unifiers, Maude would not stop because it would keep trying
to generate a second unifier for a unification problem that has only one unifier, without knowing
that it could stop. This differs from the incremental generation of variants (Section 14.6), where
we can incrementally increase the bound even if the theory does not have the finite variant
property and Maude will always stop with a new variant. The problem here is due to the fact
that Maude needs to compute the set of variants before computing the set of unifiers; when
the equational theory does not have the finite variant property, such a set of variants can be
infinite and Maude is not able to complete this computation, even when the user provides a
bound, because such a bound refers to the number of requested unifiers, but not to the number
of variants, which in this process is just part of the internal process for computing unifiers. As
in the example above, this can happen even when there are no further unifiers.

<!-- manual page 397 -->
14.12. VARIANT UNIFICATION IN INCOMPLETE UNIFICATION EXAMPLES 385
14.12 Variant unification in incomplete unification exam-
ples
Similarly to the incomplete variant generation in Section 14.7, we can have variant unification
calls that cannot provide a finitary set of most general unifiers. Let us consider again the
VARIANT-UNIFICATION-ASSOC theory of Section 14.7.
We can force unification calls, within variant unification, to an associative unification
problem with an infinite set of most general unifiers, e.g., the previously described associa-
tive unification problem N:Nat:L:NList=? L:NList:N;Nat with the family of substitutions
{L:NList →N:Nat :···: N:Nat}:
Maude> variant unify in VARIANT-UNIFICATION-ASSOC :
head(L:NList) =? last(L:NList) /\
prefix(L:NList) =? tail(L:NList) .
Warning: Unification modulo the theory of operator _:_ has encountered
an instance for which it may not be complete.
Unifier 1
L:NList --> %1:Nat : %1:Nat : %1:Nat
Unifier 2
L:NList --> %1:Nat : %1:Nat
No more unifiers.
Warning: Some unifiers may have been missed due to incomplete
unification algorithm(s).
Notice that the unification problem head(L) =? last(L) /\ prefix(L) =? tail(L) has the
same solutions as the unification problem N : L =? L : N.
14.13 The variant match command
Given a module ⟨ModId⟩, of the general form mod (Σ,G∪E∪Ax,R) endm where (Σ,E∪Ax)
satisfiestherequirementsofSection14.3andhasalsothefinitevariantproperty,Maudeprovides
a command for E∪Ax-equational matching based on variant generation of the form:
variant match [ n ] in ⟨ModId⟩ :
⟨Term-1⟩ <=? ⟨Term’-1⟩ /\ ... /\ ⟨Term-k⟩ <=? ⟨Term’-k⟩.
where k≥1; nis an optional argument providing a bound on the number of matches requested
and, as usual, if no module is mentioned, the current module is used.
Variant matching works like variant unification, except that the righthand side of each
matching problem is considered as a ground term, with variables treated as constants. Op-
erationally, a slightly different algorithm to that for variant unification is used and variant
generation is never applied to the righthand sides of the matching problems.
Consider again the module VARIANT-VENDING-MACHINE introduced in Section 14.4. We
can ask whether a configuration containing at least two quarters can match a configuration
containing at least one dollar, by invoking the following command:
Maude> variant match in VARIANT-VENDING-MACHINE :
< q q X:Marking > <=? < $ Y:Marking > .

<!-- manual page 398 -->
386 CHAPTER 14. VARIANTS AND VARIANT UNIFICATION
Matcher 1
X:Marking --> q q Y:Marking

