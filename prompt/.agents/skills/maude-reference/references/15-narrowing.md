# Chapter 15: Narrowing (manual pages 399-424)

<!-- manual page 399 -->
Chapter 15
Narrowing
15.1 Introduction
Narrowing is a generalization of term rewriting that allows logical variables in terms (as in logic
programming) and replaces pattern matching by unification in order to symbolically evaluate
these terms with given rewrite rules. Narrowing is a simple, precise answer to the question:
Given a term t with variables x1,...,xn, what are the most general instances of t
that can be rewritten by the given rules?
For example, t can be n+ m, and the rewrite rules can be 0 + y→y and s(x) + y→s(x+ y).
The term n+ m cannot be rewritten because of the variable n and finding the most general
instances becomes interesting; note that if + is commutative, then either n or m stop t from
being rewritten.
Narrowing was originally introduced as a mechanism for solving equational unification prob-
lems [64]. It was later generalized to solve the more general problem of symbolic reacha-
bility [100]. The narrowing mechanism has a number of important applications, including
automated proofs of termination [6], execution of functional-logic programming languages
[67, 32, 70, 109, 92], partial evaluation [3], verification of cryptographic protocols [100], and
equational unification [74], to mention just a few.
At each rewriting step one must choose which subterm of the subject term and which rule of
the specification are going to be considered. Similarly, at each narrowing step one must choose
which subterm of the subject term, which rule of the specification, and which instantiation
on the variables of the subject term and the rule’s lefthand side are going to be considered.
The narrowing relation is formally defined as follows. Let R= (Σ,E ∪Ax,R) be an order-
sorted rewrite theory where R is a set of unconditional rewrite rules specified with the rl
keyword, E is a set of unconditional equations specified with the eq keyword, and Axis a set of
commonly occurring axioms declared in Maude as equational attributes (see Section 4.4.1) such
that an E∪Ax-unification procedure is available in Maude.1 Let CSUE∪Ax(u= u′) provide2
a finitary and complete set of unifiers for any pair of terms u,u′ with the same top sort. The
1 E∪Ax-unification is available via the variant-based equational unification explained in Section 14.9. Note,
however, that Maude will only perform E∪Ax-unification with those equations E that have been declared with
the variant attribute. In particular, if no equation in E has the variant attribute, Maude will only perform
the Ax-unification explained in Section 13.4.
2Sections 13.4 and 14.9 provide a command for computing minimal sets of, respectively, unifiers modulo
axioms or modulo a convergent equational theory. Narrowing with rules modulo an equational theory has been
extended to use a minimal set of unifiers.
387

<!-- manual page 400 -->
388 CHAPTER 15. NARROWING
R,E∪Ax-narrowing relation on TΣ(X) is defined as tYσ,p,R,E∪Axt′(orYσ when p,R,E∪Ax
are understood) if there is a non-variable position pof t, a (possibly renamed) rule l→r in R,
and a unifier σ∈CSUE∪Ax(t|p = l) such that t′
= σ(t[r]p). We denote by tY+
σ,R,E∪Ax t′(resp.
tY∗
σ,R,E∪Ax t′)thetransitive(resp.reflexive-transitive)closureofthenarrowingrelation, where
σ is obtained as the composition of the substitutions for each narrowing step in the sequence.
The difference between a rewriting step and a narrowing step is that in both cases we use
a rewrite rule l →r to rewrite t at a position p in t, but narrowing unifies the lefthand side l
and the chosen subject term t|p before actually performing the rewriting step. Also, narrowing
is usually3 restricted to non-variable positions of t, whereas rewriting does not require such a
restriction.
Considerthefollowingsystemmoduledefiningtheadditionfunction_+_onnaturalnumbers
built from 0 and s:
mod NAT-NARROWING is
sort Nat .
op 0 : -> Nat [ctor] .
op s : Nat -> Nat [ctor] .
op _+_ : Nat Nat -> Nat .
vars X Y : Nat .
rl [base] : 0 + Y => Y .
rl [ind] : s(X) + Y => s(X + Y) .
endm
Consider the term X + s(0) and the two rules base and ind. Narrowing will instantiate
variable X with 0 and s(X’) respectively in order to be able to apply each of these rules, i.e.,
the following two narrowing steps are generated:
X + s(0)Y{X→0},base s(0)
X + s(0)Y{X→s(#1:Nat)},ind s(#1:Nat + s(0))
Note that, for simplicity, we show only the bindings of the unifier that affect the input term.
There are infinitely many narrowing derivations starting at the input expression X + s(0) (at
each step the reduced subterm is underlined):
1. X + s(0)Y{X→0},base s(0)
2. X + s(0)Y{X→s(#1:Nat)},ind s(#1:Nat + s(0))
Y{#1:Nat→0},base s(s(0))
3. X + s(0)Y{X→s(#1:Nat)},ind s(#1:Nat + s(0))
Y{#1:Nat→s(#2:Nat)},ind s(s(#2:Nat + s(0)))
Y{#2:Nat→0},base s(s(s(0)))
and so on.
The following infinite narrowing derivation resulting from applying rule ind infinitely many
times can also be shown:
X + s(0)Y{X→s(#1:Nat)},ind s(#1:Nat + s(0))
Y{#1:Nat→s(#2:Nat)},ind s(s(#2:Nat + s(0)))
...
Y{#2:Nat→s(#3:Nat)},ind s(s(s(#3:Nat + s(0))))
3The paramodulation inference rule used in paramodulation-based theorem proving [102] is similar to nar-
rowing and does not require non-variable positions.

<!-- manual page 401 -->
15.2. APPLICATIONS 389
15.2 Applications
The classical application of narrowing modulo an equational theory is to perform E ∪Ax-
unification by⃗
E,Ax-narrowing when the equations E are oriented into sort-decreasing, con-
fluent, terminating, and coherent modulo Ax rules⃗
E. Indeed the variant-based equational
order-sorted unification algorithm of Section 14.8 is based on an E,Ax-narrowing strategy,
called folding variant narrowing [62], that terminates when E∪Axhas the finite variant prop-
erty [30], even though full E,Ax-narrowing typically does not terminate when Axcontains AC
axioms [30, 62].
Instead, when the rules Rare understood as transition rules, a completely different applica-
tion of R,E∪Ax-narrowing is that of symbolic reachability analysis [100]. Specifically, we will
consider the case of transition systems specified by order-sorted rewrite theories of the form
R= (Σ,E∪Ax,R) where: (i) E∪Ax has a finite and complete E∪Ax-unification algorithm
(see the requirements of Section 14.1), and (ii) the transition rules R are E∪Ax-coherent and
topmost (so that rewriting is always done at the top of the term). Then, narrowing is a complete
deductive method [100] for symbolic reachability analysis, that is, for solving existential queries
of the form ∃xt(x) →∗t′(x) in the sense that the formula holds for Riff there is a narrowing
sequence tY∗
R,E∪Ax u such that u and t′have an E∪Ax-unifier.
Furthermore, in symbolic reachability analysis we may be interested in verifying properties
more general than existential questions of the form ∃X t−→∗t′. One can also generalize the
above reachability question to questions of the form R,t|= φ, with φa temporal logic formula.
The papers [59, 7] show how narrowing can be used (again, both at the level of transitions
with rules R and at the level of equations E) to perform logical model checking to verify such
temporal logic formulas; this is a kind of symbolic model checking not in the binary decision
diagram sense of “symbolic,” which still remains finite-state, but in a much more general sense
in which possibly infinite sets of states are finitely described by patterns with logical variables.
Two distinctive features are: (i) the term tdoes not describe a single initial state, but a possibly
infinite set of instances of t (i.e., a possibly infinite set of initial states4); and (ii) the set of
reachable states does not have to be finite. Therefore, standard model-checking techniques may
not be usable, because of a possible double infinity: in the number of initial states, and in the
number of states reachable from each of those initial states.
15.3 Completeness of narrowing
Due to nontermination, narrowing behaves as a semi-decision procedure for equational unifica-
tion and for reachability analysis in a wide variety of theories. However, for some particular
subject terms narrowing may terminate, providing a complete set of solutions. For instance, in
the Maude module NAT-NARROWING above, narrowing computes the solution {X →s(Y)}for the
reachability problem ∃X,Y 0 + X →∗s(Y) and it terminates with no more solutions. Instead,
for the reachability problem ∃X,Z X + s(0) →∗ s(s(Z)), narrowing computes the solution
{X →s(0), Z →0}but it cannot terminate because of the above infinite narrowing sequence
using ind. Moreover, narrowing cannot prove that the reachability problem ∃X X + s(0) →∗0
does not have a solution, again because of the above infinite narrowing sequence using ind.
Note that for any narrowing sequence t Y∗
σ,R,E∪Ax s, we have a corresponding rewrite
sequence σ(t) →∗
R/E∪Ax s′, where, by definition, the rewrite relation →R/E∪Ax is the relation
composition: →R/E∪Ax=def (=E∪Ax); →R; (=E∪Ax), i.e., the transition relation between states
4In general, a set of initial states may not be describable by a single pattern t, but may have a description as
a finite collection of patterns t1,...,tn. There is no problem in handling this more general case: the narrowing-
based symbolic model checking can just take a tuple ⟨t1,...,tn⟩.

<!-- manual page 402 -->
390 CHAPTER 15. NARROWING
in the given rewrite theory R= (Σ,E∪Ax,R). However, only under appropriate conditions
is narrowing complete as an equational unification algorithm [74, 75], or as a procedure to
solve reachability problems [100]. That is, given a reachability problem5 ∃x1,...,xk : s→∗t,
completeness of narrowing for reachability analysis means that for each possible substitution ρ
suchthatρ(s) →∗
R/E∪Ax ρ(t), andforallthesubstitutionsσ1,...,σn,..., providedbynarrowing
from s, there is an index iand a term ui such that, if sYσ1,R,E∪Ax u1···ui−1 Yσi,R,E∪Ax ui,
thenthereisanE∪Ax-unifierτ oftheequationui = t, andthereforethereisarewritesequence
τ(σ(s)) →∗
R/E∪Ax τ(ui), showing that an instance of s reaches an instance of t modulo the
equations E∪Ax. Essentially, completeness holds under the following conditions:
1. for(R,E∪Ax)-normalizedsubstitutionsρabove[100](astrongerconditionis(R∪E,Ax)-
normalized substitutions);
2. for topmost rewrite theories6;
3. for right-linear theories and linear reachability goals7;
4. inparticularfortheoriesthatareconfluent, terminating, andcoherentmoduloaxiomsAx,
as the equational theories in Maude functional modules with such properties restricted to
unconditional equations.
15.4 Narrowing with simplification
As pointed out in Footnote 1, given a rewrite theory R= (Σ,E∪G∪Ax,R), where Axare the
axioms, and E∪G the equations orientable as rewrite rules, if only the equations E have the
variant attribute, Maude will not perform narrowing with R modulo E∪G∪Ax, but only
modulo E ∪Ax. This gives Maude users more flexibility, since the equations G may play a
useful, auxiliary role in the applications they have in mind. In particular, a concrete, practical
reason for using the decomposition E ∪G is that E ∪Ax may be FVP, but G may not be,
so that variant E∪Ax-unification is finitary, whereas variant E∪G∪Ax-unification will be
infinitary and undecidable. Note that the standard case of narrowing with Rmodulo E∪G∪Ax
is just the special case G= ∅, which may be particularly attractive if E∪Axis FVP. The basic
intuition is that, in the most general version of narrowing supported by Maude that we now
explain, we narrow with R modulo E∪Ax, and we normalize with oriented equations⃗
E∪⃗
G
modulo Ax.
Each equation in E (used for variant computation) must include the variant attribute.
Note that equations in G do not contain the variant attribute and do not have any restric-
tion, i.e., they can be conditional equations, with the owise attribute, etc. Each narrowing
step of the form tYσ,p,R,E∪Axt′ is followed by simplification using the relation →!
G∪E,Ax,
i.e., the combined relation (Yσ,p,R,E∪Ax; →!
G∪E,Ax) is defined as tYσ,p,R,E∪Ax; →!
G∪E,Ax t′′
iff tYσ,p,R,E∪Axt′
, t′ →∗
G∪E,Ax t′′, and t′′ is G∪E,Ax-irreducible. Note that this combined
relation (Yσ,p,R,E∪Ax; →!
G∪E,Ax) may be incomplete, i.e., given a reachability problem of the
form t→∗t′and a solution σ (i.e., σ(t) →∗
R,G∪E∪Ax σ(t′)), the relationYσ,p,R,E∪Ax; →!
G∪E,Ax
may not be able to find a more general solution. The reason is that the equations G should
also be executed by narrowing instead of rewriting to ensure completeness under appropriate
5Equational unification can be represented in terms of a reachability problem [75] and, therefore, we do not
consider completeness results for equational unification here; see [4] for a survey on termination of narrowing
and completeness results for equational unification and reachability problems in the free case.
6That is, theories where every rewrite step is performed only at the top position of the term. In such theories
completeness can be simplified as follows: given ∃x1,...,xk s→∗t, for each ρ such that ρ(s) →∗
R,Ax ρ(t), and
for all the substitutions σ1,...,σn,... provided by narrowing from s, there is an index i and a substitution τ
such that ρ=Ax σi; τ.
7A reachability goal of the form ∃x1,...,xk s→∗t is linear if s is linear and s and t do not have variables
in common.

<!-- manual page 403 -->
15.5. THEORIES SUPPORTED FOR NARROWING REACHABILITY 391
conditions (see [100] and Section 15.3). However, the combination of narrowing using rules,
equations, and axioms with simplification using additional equations can be quite helpful to
allow built-in Maude functions such as addition or multiplication, which cannot be executed
by narrowing in their predefined form. It can also be useful in other applications where specific
combinations of narrowing and simplification are needed.
15.5 Theories supported for narrowing reachability
The narrowing relation is defined on top of the order-sorted variant-based unification procedure
described in Section 14.8. In order to avoid clashing of algorithms and notions, we have decided
that the rules used for narrowing should be identifiable, as the equations for variant unification,
and clearly distinguished from standard rules in Maude. For this purpose we have defined a
new attribute for rules: the keyword narrowing.
Let mod (Σ,G∪E∪Ax,R∪H) endm be an order-sorted system module where Ris a set of
rewrite rules specified with the rl keyword and the attribute narrowing, H are the remaining
rewrite rules specified with the rl or crl keywords but without the attribute narrowing, Ax
is a set of commonly occurring axioms (declared in Maude as equational attributes, see Sec-
tion 4.4.1), E is a set of equations specified with the eq keyword and the attribute variant
such that (Σ,E ∪Ax) satisfies the restrictions mentioned in Section 14.8, and G are the re-
maining equations specified with the eq or ceq keywords but without the attribute variant.
Furthermore, the rules R must satisfy the following extra conditions:
• R cannot contain conditional rules specified with the crl keyword.
• The lefthand side of a rule in R cannot be a variable. If a variable is needed, one should
instead specify a new kind with an extra unary symbol grabbing the whole system’s state
(which would before have been matched by a single variable lefthand side). In this way,
the problem of having a variable lefthand side can often be solved.
• The rules in R must be explicitly Ax-coherent (see Section 5.3 for system modules and
[97] for coherence details).
We recall again that the rules H are disregarded for narrowing modulo E ∪Ax, and the
oriented equations Gare disregarded for folding variant narrowing modulo Axin the associated
task of variant unification. However, the equations G are applied for simplification after each
narrowing step (see Section 15.4), as it is performed in Maude for rewriting. Note, again, that
this combination of one narrowing step followed by equational simplification is not complete. A
full treatment of rules, equations, and axioms for narrowing is outside the scope of the present
implementation and is left for future work.
Furthermore, frozen arguments (see Section 4.4.9) are allowed for narrowing. , They are
given the standard meaning of not allowing any narrowing step below such frozen arguments,
just as in the context-sensitive narrowing of [82].
Finally, we do not consider any narrowing strategy at all for solving reachability problems,
i.e., all positions in a term with an admissible R,E∪Ax-narrowing step are explored.
15.6 The vu-narrow command
Givenasystemmodule⟨ModId⟩,theusercangivetoMaudeanarrowing-basedsearchcommand
of the form (the prefix vu denotes that it uses variant-based unification at each step):
{fold,vfold,path} vu-narrow {filter,delay} [ n, m ] in ⟨ModId⟩ :
⟨Term-1⟩ ⟨SearchArrow⟩ ⟨Term-2⟩.
where

<!-- manual page 404 -->
392 CHAPTER 15. NARROWING
• {fold} (or {vfold}) are optional and are explained in Section 15.6.3;
• {path} is optional and is explained in Section 15.6.5;
• {filter,delay} are optional and imply that filtered variant unification of Section 14.9
is invoked;
• n is an optional argument providing a bound on the number of desired solutions;
• m is another optional argument stating the maximum depth of the search;
• the module ⟨ModId⟩where the search takes place can be omitted;
• ⟨Term-1⟩is the starting term, which typically will contain variables (Section 15.6.6 de-
scribes how to perform narrowing from a disjunction of initial terms instead of one single
term);
• ⟨Term-2⟩is the pattern that has to be reached, which may share variables with the
starting term (note that terms in the narrowing sequence will be equationally unified with
this target pattern, in contrast to the search command of Section 5.4.3 that performs
only matching);
• ⟨SearchArrow⟩is an arrow indicating the form of the narrowing proof from ⟨Term-1⟩
until ⟨Term-2⟩:
– =>1 means a narrowing proof consisting of exactly one step,
– =>+ means a narrowing proof consisting of one or more steps,
– =>* means a proof consisting of none, one, or more steps, and
– =>! indicates that only narrowing sequences ending in terms describing sets of final
states are allowed. Such terms describing sets of final states are called strongly
irreducible in the sense that they cannot be further narrowed; note that this is
stronger than requiring states that cannot be rewritten as in the search command
of Section 5.4.3.
The one step arrow =>1 is an abbreviation of the one-or-more steps arrow =>+ with the
depth bound m set to 1.
Consider, for example, the following new version of the vending machine to buy apples (a)
or cakes (c) with dollars ($) and/or quarters (q). The reader can check that the only difference
with the VARIANT-VENDING-MACHINE module in Section 14.4 is the addition of the narrowing
attribute to the rules.
mod NARROWING-VENDING-MACHINE is
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
rl [buy-c] : < M $ > => < M c > [narrowing] .
rl [buy-a] : < M $ > => < M a q > [narrowing] .
eq [change] : q q q q M = $ M [variant] .
endm
We can use the narrowing search command to answer the question:
Is there any combination of one or more coins that returns exactly an apple and a
cake?

<!-- manual page 405 -->
15.6. THE VU-NARROW COMMAND 393
This can be done by searching for states that are reachable from a term < M:Money > and
match the desired pattern at the end.
Maude> vu-narrow [1] in NARROWING-VENDING-MACHINE : < M:Money > =>* < a c > .
Solution 1
state: < a c #1:Money >
accumulated substitution:
M:Money --> $ q q q #1:Money
variant unifier:
#1:Money --> empty
Maude can now show the narrowing trace associated to each solution (see Section 15.6.5 below),
but it is easy to see that one application of the rule buy-a happened after (or before) one
application of the rule buy-c.
The narrowing-based search returns the substitution accumulated along the narrowing se-
quence and the variant unifier resulting from the unification of the target term and the last
expression in the narrowing sequence. Indeed both unifiers must be combined by hand in order
to have an actual solution to the symbolic reachability problem, as shown for the previous
example.
Solution 1
state: < a c >
accumulated substitution:
M:Money --> $ q q q
Note that
__
is an ACU symbol and that such an ACU symbol appears in the equation change,
disallowing the basic narrowing strategy [74] to be used for equational unification and requiring
the folding variant narrowing [62] to be used for equational unification.
We have restricted the previous reachability problem to just one solution. Narrowing does
not terminate for this reachability problem even though the above solution is indeed the only
solution (modulo the order of the two applications of buy-c and buy-a one after the other).
The problem is that narrowing follows a breadth-first exploration and does not stop until the
whole narrowing tree demanded by the search command is created, even though this infinite
search may not yield any further solutions. The very same problem happens for the standard
search command (see Section 5.4.3). If we increase the depth of the narrowing tree, we can
experimentally observe that there are no more solutions than the one shown before.
Maude> vu-narrow [,3] in NARROWING-VENDING-MACHINE : < M:Money > =>* < a c > .
Solution 1
state: < a c #1:Money >
accumulated substitution:
M:Money --> $ q q q #1:Money
variant unifier:
#1:Money --> empty
The command show frontier states outputs the frontier (open leaves) of the narrowing-
based reachability graph. Note that this command is not available for the search command.
For example, we can type this frontier command after the previous vu-narrow command to see
the leaves for depth 3; unfortunately, state identifiers are not shown.
Maude> show frontier states .
< c c c @1:Money > \/
< q a c c @1:Money > \/

<!-- manual page 406 -->
394 CHAPTER 15. NARROWING
< q a c c @1:Money > \/
< a c c #1:Money > \/
< q q a a c @1:Money > \/
< q a a c #1:Money > \/
< q a c c @1:Money > \/
< a c c #1:Money > \/
< q q a a c @1:Money > \/
< q a a c #1:Money > \/
< a c c @1:Money > \/
< q a a c @1:Money > \/
< q q a a c @1:Money > \/
< a a c #1:Money > \/
< q q q a a a @1:Money > \/
< q a a a #1:Money > \/
< q a a c @1:Money > \/
< a a c %1:Money > \/
< q q a a a @1:Money > \/
< q a a a %1:Money >
In the previous reachability problem we can change the arrow =>* for reachability in zero
or more steps by the arrow =>! for reachability in zero or more steps including only states that
cannot be narrowed any more.
Maude> vu-narrow [,3] in NARROWING-VENDING-MACHINE : < M:Money > =>! < a c > .
No solution.
And surprisingly we do not find the previous solution. The reason is that the transition rules
of the vending machine are not terminating for narrowing and for rewriting, so it is impossible
to find a state that cannot be narrowed any more. However, if we replace the variable M:Money
by variables of sort Coin, we are able to find appropriate solutions; that is, we formulate the
following interesting question:
Is there any combination of four coins that returns an apple and a cake and is such
that some extra money is left but that extra money cannot be used to buy anything
else?
The fact that some money may be left is characterized by including a variable of sort Money in
the final state, and the fact that nothing else can be bought is characterized by using the =>!
arrow instead of =>*.
Maude> vu-narrow in NARROWING-VENDING-MACHINE :
< C1:Coin C2:Coin C3:Coin C4:Coin > =>! < M:Money a c > .
Solution 1
state: < a c >
accumulated substitution:
C1:Coin --> q
C2:Coin --> q
C3:Coin --> q
C4:Coin --> $
variant unifier:
M:Money --> empty
Solution 2

<!-- manual page 407 -->
15.6. THE VU-NARROW COMMAND 395
state: < a c >
accumulated substitution:
C1:Coin --> q
C2:Coin --> q
C3:Coin --> $
C4:Coin --> q
variant unifier:
M:Money --> empty
Solution 3
state: < a c >
accumulated substitution:
C1:Coin --> q
C2:Coin --> $
C3:Coin --> q
C4:Coin --> q
variant unifier:
M:Money --> empty
Solution 4
state: < a c >
accumulated substitution:
C1:Coin --> $
C2:Coin --> q
C3:Coin --> q
C4:Coin --> q
variant unifier:
M:Money --> empty
No more solutions.
Indeed, the show frontier states command shows that there are no open leaves because the
narrowing tree is terminating.
Maude> show frontier states .
*** frontier is empty ***
Another point of interest is the occurrence of variables of the form #n:Sort or %n:Sort,
which are called fresh and are described in Chapter 13. Unification modulo axioms usually
introduces fresh variables; furthermore, narrowing introduces many fresh variables because the
rule applied at each narrowing step is appropriately renamed so that no variable is shared
by it and the current term. Indeed, the standard solution used in logic and functional-logic
programming language implementations is to use a counter along each narrowing derivation to
ensure that fresh variables have never been used previously in that narrowing derivation. This
method is called standardized apart [5] and it is summarized by saying that a new version of a
rule, equation or axiom is always generated before being used for unification or narrowing by
renaming variables according to the counter, which is incremented afterwards. This method
makes the result of a computation independent of the choice of variable names.
15.6.1 Variant unification options
Sections 13.4 and 14.9 provide a command for computing minimal sets of, respectively, unifiers
modulo axioms or modulo a convergent equational theory. Narrowing with rules modulo an
equational theory has also been extended to use a minimal set of unifiers. The filter flag

<!-- manual page 408 -->
396 CHAPTER 15. NARROWING
indicates that filtering is activated but it can be done either during equational unification (the
delay flag is not included) or after all equational unifiers have been computed (the delay flag
is included); the latter being the method used in the filtered variant unify command of
Section 14.9.
For instance, we can use the narrowing search command to answer the question:
Can I get an apple and a cake starting with the exact money?
The answer should be an obvious yes but the symbolic reachability question gets into too many
paths.
Maude> vu-narrow [, 2] in NARROWING-VENDING-MACHINE :
< $ q q q M1:Money > =>* < a c M2:Money > .
Solution 1
state: < $ a c %1:Money >
accumulated substitution:
M1:Money --> $ %1:Money
variant unifier:
M2:Money --> $ @1:Money
%1:Money --> @1:Money
...
Solution 9
state: < $ a c #1:Money >
accumulated substitution:
M1:Money --> q q q q #1:Money
variant unifier:
M2:Money --> $ @1:Money
#1:Money --> @1:Money
No more solutions.
rewrites: 34 in 9ms cpu (10ms real) (3664 rewrites/second)
We can ask Maude to filter unifiers at each narrowing step, and the branching is, obviously,
smaller.
Maude> vu-narrow {filter} [, 2] in NARROWING-VENDING-MACHINE :
< $ q q q M1:Money > =>* < a c M2:Money > .
Solution 1
state: < $ a c %1:Money >
accumulated substitution:
M1:Money --> $ %1:Money
variant unifier:
M2:Money --> $ @1:Money
%1:Money --> @1:Money
Solution 2
state: < q a c #1:Money >
accumulated substitution:
M1:Money --> q #1:Money
variant unifier:
M2:Money --> q @1:Money

<!-- manual page 409 -->
15.6. THE VU-NARROW COMMAND 397
#1:Money --> @1:Money
Solution 3
state: < a c %1:Money >
accumulated substitution:
M1:Money --> %1:Money
variant unifier:
M2:Money --> @1:Money
%1:Money --> @1:Money
No more solutions.
But the first solution is actually redundant, i.e., we can ask Maude to filter unifiers only after
all have been generated, which is more expensive but ensures the minimal set of most general
variant unifiers.
Maude> vu-narrow {filter,delay} [, 2] in NARROWING-VENDING-MACHINE :
< $ q q q M1:Money > =>* < a c M2:Money > .
Solution 1
state: < q a c #1:Money >
accumulated substitution:
M1:Money --> q #1:Money
variant unifier:
M2:Money --> q @1:Money
#1:Money --> @1:Money
Solution 2
state: < a c %1:Money >
accumulated substitution:
M1:Money --> %1:Money
variant unifier:
M2:Money --> @1:Money
%1:Money --> @1:Money
No more solutions.
Note that the simplest path is the second solution, since all the necessary money was given.
15.6.2 Filtered vs. unfiltered symbolic reachability analysis
The vu-narrow {filter,delay} command can achieve dramatic state space reductions over
the previous vu-narrow command by filtering E∪B-unifiers. This is illustrated in this section
by a simple rewrite theory describing a cryptographic protocol inspired by the strand space
model [63], which is the basis for the Maude-NPA protocol analyzer [57, 58]. The protocol ap-
peared in [54, 60]. The cryptographic function’s equational properties are those of the following
exclusive-or FVP theory:
fmod EXCLUSIVE-OR is
sort XOR .
op mt : -> XOR .
op _*_ : XOR XOR -> XOR [assoc comm] .
vars X Y Z U V : [XOR] .
eq [nilp] : X * X = mt [variant] .
eq [nilp-Coh] : X * X * Z = Z [variant] .

<!-- manual page 410 -->
398 CHAPTER 15. NARROWING
eq [id] : endfm
X * mt = X [variant] .
The strand space syntax is given in a general form.
mod PROTOCOL-SYNTAX is
sorts Name Nonce Fresh Msg .
subsort Name Nonce < Msg .
ops a b i : -> Name . --- Alice Bob Intruder
op n : Name Fresh -> Nonce . --- Nonces
ops r1 r2 r3 : -> Fresh .
op pk : Name Msg -> Msg . --- Public encryption
sort SMsg .
ops + - : Msg -> SMsg .
sort SMsgList . subsort SMsg < SMsgList .
op nil : -> SMsgList .
op _,_ : SMsgList SMsgList -> SMsgList [assoc] .
sort Strand .
op [_|_] : SMsgList SMsgList -> Strand .
sort StrandSet .
subsort Strand < StrandSet .
op mt : -> StrandSet .
op _&_ : StrandSet StrandSet -> StrandSet [assoc comm id: mt] .
sort IKnowledge .
op mt : -> IKnowledge .
op inI : Msg -> IKnowledge . --- Intruder knows
op nI : Msg -> IKnowledge . --- Intruder doesn’t know
op _,_ : IKnowledge IKnowledge -> IKnowledge [assoc comm id: mt] .
sort State .
op {_{_}} : StrandSet IKnowledge -> State .
endm
Theexclusive-orprotocolexampleisdefinedinthefollowingmodule,wherenoncesarecombined
using exclusive-or.
mod PROTOCOL-XOR is
extending EXCLUSIVE-OR .
extending PROTOCOL-SYNTAX .
subsort Nonce < XOR < Msg .
var IK : IKnowledge . var SS : StrandSet .
var M : Msg . vars L1 L2 : SMsgList .
--- Intruder knows receiving message of a strand
rl [recv] : { (SS & [ ( L1 , -(M)) | L2 ]) { (inI(M) , IK) } } =>
{ (SS & [ L1 | (-(M) , L2) ]) { (inI(M) , IK) } } [narrowing] .
--- Intruder learns a sending message of a strand
rl [snd] : { (SS & [ (L1 , +(M)) | L2 ]) { (inI(M) , IK) } } =>
{ (SS & [ L1 | (+(M) , L2) ]) { (nI(M) , IK) } } [narrowing] .
endm

<!-- manual page 411 -->
15.6. THE VU-NARROW COMMAND 399
Actually, due to this general strand syntax, the protocol itself is given as a term. And, due to
the fact that the rewrite rules are given in a reversed way, the initial term contains the protocol
with the vertical bars on the rightmost position, denoting a finished strand, and the intruder
knowledge with inI operators, denoting terms learned by the intruder, whereas the target term
contains the protocol with the vertical bars on the leftmost position, denoting a non-started
strand, and the intruder knowledge with nI operators, denoting terms not yet learned by the
intruder. The initial term of the narrowing-based reachability command is
{
[ nil, -(pk(a, X)), +(pk(b, n(a, r2))), -(X * n(a, r2)) | nil] &
[ nil, +(pk(a, n(b, r1))), -(pk(b, Y )), +(Y * n(b, r1)) | nil]
{ inI(X * n(a, r2)), inI(pk(b, Y )), inI(pk(a, X)) }
and the target term of the narrowing-based reachability command is
}
{
[ nil | -(pk(a, X)), +(pk(b, n(a, r2))), -(X * n(a, r2)), nil] &
[ nil | +(pk(a, n(b, r1))), -(pk(b, Y )), +(Y * n(b, r1)), nil]
{ nI(X * n(a, r2)), nI(pk(b, Y )), nI(pk(a, X)) }
}
Note that there are variables X and Y only for the unknown parts of receiving messages of Alice
and Bob. This reduces the search space considerably: it becomes a finite symbolic search space
with depth 6, so we can focus only on the difference between filtered and unfiltered variant
unification. The unfiltered symbolic reachability search command without folding is as follows:
Maude> vu-narrow
{
[ nil, -(pk(a, X:Msg)), +(pk(b, n(a, r2))), -(X:Msg * n(a, r2)) | nil] &
[ nil, +(pk(a, n(b, r1))), -(pk(b, Y:Msg)), +(Y:Msg * n(b, r1)) | nil]
{ inI(X:Msg * n(a, r2)), inI(pk(b, Y:Msg)), inI(pk(a, X:Msg)) }
}
=>* {
[ nil | -(pk(a, X:Msg)), +(pk(b, n(a, r2))), -(X:Msg * n(a, r2)), nil] &
[ nil | +(pk(a, n(b, r1))), -(pk(b, Y:Msg)), +(Y:Msg * n(b, r1)), nil]
{ nI(X:Msg * n(a, r2)), nI(pk(b, Y:Msg)), nI(pk(a, X:Msg)) }
} .
Solution 1
rewrites: 26528 in 1937ms cpu (2163ms real) (13694 rewrites/second)
state: {[nil | +(pk(a, n(b, r1))),-(pk(b, n(a, r2))),+(n(a, r2) * n(b, r1)),nil] &
[nil | -(pk(a, n(b, r1))),+(pk(b, n(a, r2))),-(n(a, r2) * n(b, r1)),nil]{nI(n(a,
r2) * n(b, r1)),nI(pk(a, n(b, r1))),nI(pk(b, n(a, r2)))}}
accumulated substitution:
Y:Msg --> mt * n(a, r2)
X:Msg --> mt * n(b, r1)
variant unifier:
Solution 2
rewrites: 26533 in 1938ms cpu (2164ms real) (13689 rewrites/second)
state: {[nil | +(pk(a, n(b, r1))),-(pk(b, n(a, r2))),+(n(a, r2) * n(b, r1)),nil] &
[nil | -(pk(a, n(b, r1))),+(pk(b, n(a, r2))),-(n(a, r2) * n(b, r1)),nil]{nI(n(a,
r2) * n(b, r1)),nI(pk(a, n(b, r1))),nI(pk(b, n(a, r2)))}}
accumulated substitution:
Y:Msg --> n(a, r2) * n(a, r2) * n(a, r2)

<!-- manual page 412 -->
400 CHAPTER 15. NARROWING
X:Msg --> n(a, r2) * n(a, r2) * n(b, r1)
variant unifier:
...
Solution 84
rewrites: 26937 in 2015ms cpu (2243ms real) (13364 rewrites/second)
state: {[nil | +(pk(a, n(b, r1))),-(pk(b, n(a, r2))),+(n(a, r2) * n(b, r1)),nil] &
[nil | -(pk(a, n(b, r1))),+(pk(b, n(a, r2))),-(n(a, r2) * n(b, r1)),nil]{nI(n(a,
r2) * n(b, r1)),nI(pk(a, n(b, r1))),nI(pk(b, n(a, r2)))}}
accumulated substitution:
Y:Msg --> n(a, r2) * n(b, r1) * n(b, r1)
X:Msg --> n(a, r2) * n(a, r2) * n(b, r1)
variant unifier:
No more solutions.
The symbolic search space is finite: 916 nodes, depth 6, and 84 solutions (all similar to the one
below).8 The filtered symbolic reachability search command without folding is as follows:
Maude> vu-narrow {filter,delay}
{
[ nil, -(pk(a, X:Msg)), +(pk(b, n(a, r2))), -(X:Msg * n(a, r2)) | nil] &
[ nil, +(pk(a, n(b, r1))), -(pk(b, Y:Msg)), +(Y:Msg * n(b, r1)) | nil]
{ inI(X:Msg * n(a, r2)), inI(pk(b, Y:Msg)), inI(pk(a, X:Msg)) }
}
=>* {
[ nil | -(pk(a, X:Msg)), +(pk(b, n(a, r2))), -(X:Msg * n(a, r2)), nil] &
[ nil | +(pk(a, n(b, r1))), -(pk(b, Y:Msg)), +(Y:Msg * n(b, r1)), nil]
{ nI(X:Msg * n(a, r2)), nI(pk(b, Y:Msg)), nI(pk(a, X:Msg)) }
} .
Solution 1
rewrites: 2197 in 129ms cpu (144ms real) (16937 rewrites/second)
state: {[nil | +(pk(a, n(b, r1))),-(pk(b, n(a, r2))),+(n(a, r2) * n(b, r1)),nil] &
[nil | -(pk(a, n(b, r1))),+(pk(b, n(a, r2))),-(n(a, r2) * n(b, r1)),nil]{nI(n(a,
r2) * n(b, r1)),nI(pk(a, n(b, r1))),nI(pk(b, n(a, r2)))}}
accumulated substitution:
Y:Msg --> mt * n(a, r2)
X:Msg --> mt * n(b, r1)
variant unifier:
No more solutions.
The symbolic search space is also finite but much smaller: 20 nodes, depth 6, and just one
solution, with X →mt * n(a,r2) and Y →mt * n(b,r1); see Section 15.6.4 below for even
bigger state space reductions. Since exclusive-or is unitary [76], each narrowing step has only
one child and, for some nodes, there are two possible narrowing steps either in Alice’s or in
Bob’s strands.
8This information can be obtained using the set verbose on . command.

<!-- manual page 413 -->
15.6. THE VU-NARROW COMMAND 401
15.6.3 Folding the narrowing space
We have motivated in the previous sections how narrowing can be used for symbolic reacha-
bility analysis [100] using the vu-narrow command. However, narrowing builds a reachability
tree rather than a reachability graph, which is what the search command of Section 5.4.3
does for rewriting with rules. We have endowed the vu-narrow command with two different
flags, either {fold} or {vfold}, that fold some branches of the narrowing reachability tree
in order to create a graph, in the case of fold using only axioms and in the case of vfold
using variant equations and axioms. The main difference is that the search command identi-
fies states in the rewriting-based reachability tree that are equal modulo axioms whereas the
{fold/vfold} vu-narrow command identifies states in the narrowing-based reachability tree
that are equal modulo axioms and equations under instantiation9. This is similar in nature to
the folding variant narrowing strategy [62] of the variant generation command of Chapter 14
but using rules instead of equations. Indeed, the logical model checking of [59, 7] is based
on earlier versions of this {fold/vfold} vu-narrow command. An abbreviation fvu-narrow
is provided, which expands into the full command {fold} vu-narrow but without any other
option.
Consider again the NARROWING-VENDING-MACHINE module and the following question, both
in Section 15.6.
Is there any combination of one or more coins that returns exactly an apple and a
cake?
We can repeat, with the same result, the vu-narrow command but now with the f at the
beginning.
Maude> fvu-narrow [,3] in NARROWING-VENDING-MACHINE : < M:Money > =>* < a c > .
Solution 1
state: < a c #1:Money >
accumulated substitution:
M:Money --> $ q q q #1:Money
variant unifier:
#1:Money --> empty
Afterexecutinga{fold} vu-narrowcommand(orfvu-narrow),thenewshow most general
states command outputs those states that have not been folded in the generated narrowing-
based reachability graph. For example, we can show the most general states after the previous
fvu-narrow command to see the leaves for depth 3. State identifiers are not shown, but this
may be changed in the future.
Maude> show most general states .
< #1:Money > \/
< c @1:Money > \/
< q a @1:Money > \/
< c c %1:Money > \/
< a c #1:Money > \/
< q a a #1:Money > \/
< c c c @1:Money > \/
< a c c @1:Money > \/
< a a c %1:Money > \/
< q a a a %1:Money >
9A new generated state s2 (possibly with variables) is folded into a previously generated state s1 (possibly
with variables, different from those of s2) if s2 is an instance of s1 modulo axioms and variant equations.

<!-- manual page 414 -->
402 CHAPTER 15. NARROWING
There is actually no folding here, since all the combinations for depth 3 are shown. We can
also ask for the open leaves.
Maude> show frontier states .
< c c c @1:Money > \/
< a c c @1:Money > \/
< a a c %1:Money > \/
< q a a a %1:Money >
Note that, if folding is used, the frontier will always be a subset of the most general states,
since the folded states are no longer explored by narrowing.
Moreover, producing a reachability graph instead of a reachability tree may improve the
chances of having a finite search space. If we repeat the same command but unrestricted,
Maude reports the unique solution but gets into a loop, since the fvu-narrow command cannot
produce a finite search graph. However, the reason is that we are giving a logical variable of
sort Money and any new state containing an apple or a cake cannot be an instance of the initial
state. Therefore, we can use a variable of a bigger sort Marking instead of Money and the
infinite narrowing-based reachability tree is folded into a finite narrowing-based reachability
graph.
Maude> fvu-narrow in NARROWING-VENDING-MACHINE : < M:Marking > =>* < a c > .
Solution 1
state: < #1:Marking >
accumulated substitution:
M:Marking --> #1:Marking
variant unifier:
#1:Marking --> a c
No more solutions.
But this is not the expected solution, since it is simply instantiating variable M:Marking to the
apple and the cake. The graph is finite, as we wanted, but there is only one state, the initial
one, and every other state is folded into it, since they are all obviously instances of it.
Whatisactuallyhappeningisthatthisspecificationofthevendingmachineisnotwellsuited
fortakingadvantageofthisfoldingnarrowingtechnique, sincefoldingimpliesthatanewstateis
an instance of a previous one and a rewrite rule where the right-hand side removes information
from the left-hand side instead of adding more information may help to fold. Consider the
following new version of the vending machine. The reader can check that the only difference
with the NARROWING-VENDING-MACHINE module in Section 15.6 is that apples and cakes are not
deposited in the bag but consumed. This is typical of a logic programming style (in the sense
of Prolog), where computed answers are used because there are no normal forms.
mod FOLDING-NARROWING-VENDING-MACHINE is
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

<!-- manual page 415 -->
15.6. THE VU-NARROW COMMAND 403
rl [buy-c] : < M $ c > => < M > [narrowing] .
rl [buy-a] : < M $ a > => < M q > [narrowing] .
eq [change] : q q q q M = $ M [variant] .
endm
We can repeat the same command and now we get the expected result, while having a finite
narrowing-based reachability graph.
Maude> fvu-narrow in FOLDING-NARROWING-VENDING-MACHINE :
< M:Marking c c > =>* < empty > .
Solution 1
state: < %1:Marking >
accumulated substitution:
M:Marking --> $ $ %1:Marking
variant unifier:
%1:Marking --> empty
No more solutions.
The graph is finite, as we wanted, since, after two narrowing steps, a state of the form
< #1:Marking > is generated and, therefore, any other further state is folded into it. Note
that a similar vu-narrow command does not stop. Note also that if we use a variable of
sort Money, then the narrowing-based reachability tree is finite, since no more coins can be
introduced, and both the vu-narrow and fvu-narrow commands stop.
Maude> vu-narrow in FOLDING-NARROWING-VENDING-MACHINE :
< M:Money c c > =>* < empty > .
Solution 1
state: < %1:Money >
accumulated substitution:
M:Money --> $ $ %1:Money
variant unifier:
%1:Money --> empty
No more solutions.
15.6.4 Reducing the symbolic reachability analysis using vfold
The difference between using the {fold} vu-narrow command and the {vfold} vu-narrow
command may not seem relevant, since variant equations are used also for simplification. Even
more, the combination with {filter,delay} may also seem not relevant, but combining all
may have a huge impact.
For example, let us consider the system module of Section 15.6.2 describing a cryptographic
protocol inspired by the strand space model. On that system module we executed the following
command (see page 399) that we are going to be adapting with different options10
.
Maude> vu-narrow
{
[ nil, -(pk(a, X:Msg)), +(pk(b, n(a, r2))), -(X:Msg * n(a, r2)) | nil] &
[ nil, +(pk(a, n(b, r1))), -(pk(b, Y:Msg)), +(Y:Msg * n(b, r1)) | nil]
{ inI(X:Msg * n(a, r2)), inI(pk(b, Y:Msg)), inI(pk(a, X:Msg)) }
10This information can be obtained using the set verbose on . command.

<!-- manual page 416 -->
404 CHAPTER 15. NARROWING
}
=>* {
[ nil | -(pk(a, X:Msg)), +(pk(b, n(a, r2))), -(X:Msg * n(a, r2)), nil] &
[ nil | +(pk(a, n(b, r1))), -(pk(b, Y:Msg)), +(Y:Msg * n(b, r1)), nil]
{ nI(X:Msg * n(a, r2)), nI(pk(b, Y:Msg)), nI(pk(a, X:Msg)) }
} .
1. The command vu-narrow has 916 states and 84 solutions in the reachability tree. It
made 26937 steps in 1322ms.
2. The command vu-narrow {filter,delay} has 20 states and 1 solution in the reacha-
bility tree, since a minimal set of most general unifiers is computed for each narrowing
step and we know that there is always only one most general unifier for the exclusive-or
theory. It made 1899 steps in 85ms.
3. A command {fold} vu-narrow has 197 states and 1 solution in the reachability graph,
since all the 84 solutions of the first command are folded into only one of them. It made
4908 steps in 268ms.
4. A command {fold} vu-narrow {filter,delay} has 19 states and 1 solution in the
reachability graph, since the computation of the minimal set of most general unifiers
already reduces the search space. It made 1892 steps in 84ms.
5. A command {vfold} vu-narrow has 68 states and 1 solution in the reachability graph,
since folding using the full exclusive-or theory instead of only the axioms is more powerful
and folds many more states. It made 1633 steps in 88ms.
6. A command {vfold} vu-narrow {filter,delay} has 19 states and 1 solution in the
reachability graph, since all the improvements are combined. It made 1892 steps in 96ms.
Clearly, the best option is to compute a minimal set of most general unifiers for each narrowing
step. However, we can see a difference between folding with the whole equational theory or
only with the axioms.
Folding does not seem to be useful because the protocol example is terminating for narrow-
ing, but this is not actually true in general. Let us add an idle transition rule to the system,
generating a self-loop for every state, as in stuttering simulations.
mod PROTOCOL-XOR-STUTTERING is
extending EXCLUSIVE-OR .
extending PROTOCOL-SYNTAX .
subsort Nonce < XOR < Msg .
var IK : IKnowledge . var SS : StrandSet .
var M : Msg . vars L1 L2 : SMsgList .
--- Intruder knows receiving message of a strand
rl [recv] : { (SS & [ ( L1 , -(M)) | L2 ]) { (inI(M) , IK) } } =>
{ (SS & [ L1 | (-(M) , L2) ]) { (inI(M) , IK) } } [narrowing] .
--- Intruder learns a sending message of a strand
rl [snd] : { (SS & [ (L1 , +(M)) | L2 ]) { (inI(M) , IK) } } =>
{ (SS & [ L1 | (+(M) , L2) ]) { (nI(M) , IK) } } [narrowing] .
--- Stuttering action
rl [idle] : { SS { IK } } => { SS { IK } } [narrowing] .
endm
Now folding becomes very useful.
1. The command vu-narrow is not able to report any solution and a depth bound does not
help, since the number of useless paths is overwhelming.
2. The command vu-narrow {filter,delay} is able to report solutions but there is an
infinite number of them due to self-loops.

<!-- manual page 417 -->
15.6. THE VU-NARROW COMMAND 405
3. A command {fold} vu-narrow has 686 states and 1 solution in the reachability graph.
It made 7557 steps in 465ms.
4. A command {fold} vu-narrow {filter,delay} has 35 states and 1 solution in the
reachability graph. It made 2664 steps in 126ms.
5. A command {vfold} vu-narrow has 148 states and 1 solution in the reachability graph.
It made 2091 steps in 119ms.
6. A command {vfold} vu-narrow {filter,delay} has 35 states and 1 solution in the
reachability graph. It made 2664 steps in 134ms.
In conclusion, combining variant-based folding with the computation of a minimal set of most
general unifiers is extremely useful.
15.6.5 Narrowing path commands
It is now possible to obtain the narrowing trace to a specific solution in a narrowing-based
reachability problem, as it happens with the search command of Section 5.4.3. However, this
is an expensive feature and the option {path} must be given at the beginning of the vu-narrow
command in order to store the appropriate information. When this option is given, the two
standard path commands can be used
show path <state number> .
show path states <state number> .
Naturally, the option path can be combined with the option fold and shown below.
For example, let us consider again the simple rewrite theory of Section 15.6.2 describing a
cryptographic protocol inspired by the strand space model. First, we add the {path} option at
the beginning of the last command of Section 15.6.4.
Maude> {vfold,path} vu-narrow {filter,delay}
{
[ nil, -(pk(a, X:Msg)), +(pk(b, n(a, r2))), -(X:Msg * n(a, r2)) | nil] &
[ nil, +(pk(a, n(b, r1))), -(pk(b, Y:Msg)), +(Y:Msg * n(b, r1)) | nil]
{ inI(X:Msg * n(a, r2)), inI(pk(b, Y:Msg)), inI(pk(a, X:Msg)) }
}
=>* {
[ nil | -(pk(a, X:Msg)), +(pk(b, n(a, r2))), -(X:Msg * n(a, r2)), nil] &
[ nil | +(pk(a, n(b, r1))), -(pk(b, Y:Msg)), +(Y:Msg * n(b, r1)), nil]
{ nI(X:Msg * n(a, r2)), nI(pk(b, Y:Msg)), nI(pk(a, X:Msg)) }
} .
Thiscommandhadonlyonemostgeneralsolutionbut, now, itshowswhichisthestateidentifier
associated to this solution. Then, we type the show path command and we get the following
output.
Maude> show path 32 .
state 0, [State]:
{[nil, +(pk(a, n(b, r1))), -(pk(b, #1:Msg)), +(#1:Msg * n(b, r1)) | nil] &
[nil, -(pk(a, #2:Msg)), +(pk(b, n(a, r2))), -(#2:Msg * n(a, r2)) | nil]
{inI(#2:Msg * n(a, r2)), inI(pk(a, #2:Msg)), inI(pk(b, #1:Msg))}}
accumulated substitution:
Y:Msg --> #1:Msg
X:Msg --> #2:Msg
===[ rl [recv] :
{SS:StrandSet & [L1:SMsgList, -(M:Msg) | L2:SMsgList]{IK:IKnowledge, inI(M:Msg)}}

<!-- manual page 418 -->
406 CHAPTER 15. NARROWING
=> {SS:StrandSet & [L1:SMsgList | -(M:Msg), L2:SMsgList]{IK:IKnowledge, inI(M:Msg)}}
[narrowing] . ]===>
variant unifier:
SS:StrandSet --> [nil, +(pk(a, n(b, r1))), -(pk(b, @1:XOR)), +(@1:XOR * n(b, r1))
| nil]
L1:SMsgList --> nil, -(pk(a, @2:XOR)), +(pk(b, n(a, r2)))
M:Msg --> @2:XOR * n(a, r2)
L2:SMsgList --> nil
IK:IKnowledge --> inI(pk(a, @2:XOR)), inI(pk(b, @1:XOR))
#1:Msg --> @1:XOR
#2:Msg --> @2:XOR
state 1, State:
{[nil, -(pk(a, @2:XOR)), +(pk(b, n(a, r2))) | -(@2:XOR * n(a, r2)), nil] &
[nil, +(pk(a, n(b, r1))), -(pk(b, @1:XOR)), +(@1:XOR * n(b, r1)) | nil]
{inI(@2:XOR * n(a, r2)), inI(pk(a, @2:XOR)), inI(pk(b, @1:XOR))}}
accumulated substitution:
Y:Msg --> @1:XOR
X:Msg --> @2:XOR
===[ rl [snd] :
{SS:StrandSet & [L1:SMsgList, +(M:Msg) | L2:SMsgList]{IK:IKnowledge, inI(M:Msg)}}
=> {SS:StrandSet & [L1:SMsgList | +(M:Msg), L2:SMsgList]{IK:IKnowledge, nI(M:Msg)}}
[narrowing] . ]===>
variant unifier:
SS:StrandSet --> [nil, -(pk(a, %1:XOR * n(b, r1))), +(pk(b, n(a, r2)))
| -(%1:XOR * n(a, r2) * n(b, r1)), nil]
L1:SMsgList --> nil, +(pk(a, n(b, r1))), -(pk(b, %1:XOR * n(a, r2)))
M:Msg --> %1:XOR * n(a, r2) * n(b, r1)
L2:SMsgList --> nil
IK:IKnowledge --> inI(pk(a, %1:XOR * n(b, r1))), inI(pk(b, %1:XOR * n(a, r2)))
@2:XOR --> %1:XOR * n(b, r1)
@1:XOR --> %1:XOR * n(a, r2)
state 5, State:
{[nil, +(pk(a, n(b, r1))), -(pk(b, %1:XOR * n(a, r2))) | +(%1:XOR * n(a, r2) *
n(b, r1)), nil] &
[nil, -(pk(a, %1:XOR * n(b, r1))), +(pk(b, n(a, r2))) | -(%1:XOR * n(a, r2) *
n(b, r1)), nil]
{inI(pk(a, %1:XOR * n(b, r1))), inI(pk(b, %1:XOR * n(a, r2))), nI(%1:XOR *
n(a, r2) * n(b, r1))}}
accumulated substitution:
Y:Msg --> %1:XOR * n(a, r2)
X:Msg --> %1:XOR * n(b, r1)
===[ rl [recv] :
{SS:StrandSet & [L1:SMsgList, -(M:Msg) | L2:SMsgList]{IK:IKnowledge, inI(M:Msg)}}
=> {SS:StrandSet & [L1:SMsgList | -(M:Msg), L2:SMsgList]{IK:IKnowledge, inI(M:Msg)}}
[narrowing] . ]===>
variant unifier:
SS:StrandSet --> [nil, -(pk(a, @1:XOR * n(b, r1))), +(pk(b, n(a, r2)))
| -(@1:XOR * n(a, r2) * n(b, r1)), nil]
L1:SMsgList --> nil, +(pk(a, n(b, r1)))
M:Msg --> pk(b, @1:XOR * n(a, r2))
L2:SMsgList --> +(@1:XOR * n(a, r2) * n(b, r1)), nil
IK:IKnowledge --> inI(pk(a, @1:XOR * n(b, r1))), nI(@1:XOR * n(a, r2) * n(b, r1))

<!-- manual page 419 -->
15.6. THE VU-NARROW COMMAND 407
%1:XOR --> @1:XOR
state 12, State:
{[nil, +(pk(a, n(b, r1))) | -(pk(b, @1:XOR * n(a, r2))), +(@1:XOR * n(a, r2) *
n(b, r1)), nil] &
[nil, -(pk(a, @1:XOR * n(b, r1))), +(pk(b, n(a, r2))) | -(@1:XOR * n(a, r2) *
n(b, r1)), nil]
{inI(pk(a, @1:XOR * n(b, r1))), inI(pk(b, @1:XOR * n(a, r2))), nI(@1:XOR *
n(a, r2) * n(b, r1))}}
accumulated substitution:
Y:Msg --> @1:XOR * n(a, r2)
X:Msg --> @1:XOR * n(b, r1)
===[ rl [snd] :
{SS:StrandSet & [L1:SMsgList, +(M:Msg) | L2:SMsgList]{IK:IKnowledge, inI(M:Msg)}}
=> {SS:StrandSet & [L1:SMsgList | +(M:Msg), L2:SMsgList]{IK:IKnowledge, nI(M:Msg)}}
[narrowing] . ]===>
variant unifier:
SS:StrandSet --> [nil, +(pk(a, n(b, r1)))
| -(pk(b, n(a, r2))), +(n(a, r2) * n(b, r1)), nil]
L1:SMsgList --> nil, -(pk(a, n(b, r1)))
M:Msg --> pk(b, n(a, r2))
L2:SMsgList --> -(n(a, r2) * n(b, r1)), nil
IK:IKnowledge --> inI(pk(a, n(b, r1))), nI(n(a, r2) * n(b, r1))
@1:XOR --> (mt).XOR
state 22, State:
{[nil, +(pk(a, n(b, r1))) | -(pk(b, n(a, r2))), +(n(a, r2) * n(b, r1)), nil] &
[nil, -(pk(a, n(b, r1))) | +(pk(b, n(a, r2))), -(n(a, r2) * n(b, r1)), nil]
{inI(pk(a, n(b, r1))), nI(n(a, r2) * n(b, r1)), nI(pk(b, n(a, r2)))}}
accumulated substitution:
Y:Msg --> mt * n(a, r2)
X:Msg --> mt * n(b, r1)
===[ rl [recv] :
{SS:StrandSet & [L1:SMsgList, -(M:Msg) | L2:SMsgList]{IK:IKnowledge, inI(M:Msg)}}
=> {SS:StrandSet & [L1:SMsgList | -(M:Msg), L2:SMsgList]{IK:IKnowledge, inI(M:Msg)}}
[narrowing] . ]===>
variant unifier:
SS:StrandSet --> [nil, +(pk(a, n(b, r1)))
| -(pk(b, n(a, r2))), +(n(a, r2) * n(b, r1)), nil]
L1:SMsgList --> nil
M:Msg --> pk(a, n(b, r1))
L2:SMsgList --> +(pk(b, n(a, r2))), -(n(a, r2) * n(b, r1)), nil
IK:IKnowledge --> nI(n(a, r2) * n(b, r1)), nI(pk(b, n(a, r2)))
state 28, State:
{[nil | -(pk(a, n(b, r1))), +(pk(b, n(a, r2))), -(n(a, r2) * n(b, r1)), nil] &
[nil, +(pk(a, n(b, r1))) | -(pk(b, n(a, r2))), +(n(a, r2) * n(b, r1)), nil]
{inI(pk(a, n(b, r1))), nI(n(a, r2) * n(b, r1)), nI(pk(b, n(a, r2)))}}
accumulated substitution:
Y:Msg --> mt * n(a, r2)
X:Msg --> mt * n(b, r1)
===[ rl [snd] :
{SS:StrandSet & [L1:SMsgList, +(M:Msg) | L2:SMsgList]{IK:IKnowledge, inI(M:Msg)}}
=> {SS:StrandSet & [L1:SMsgList | +(M:Msg), L2:SMsgList]{IK:IKnowledge, nI(M:Msg)}}

<!-- manual page 420 -->
408 CHAPTER 15. NARROWING
[narrowing] . ]===>
variant unifier:
SS:StrandSet --> [nil | -(pk(a, n(b, r1))),
+(pk(b, n(a, r2))), -(n(a, r2) * n(b, r1)), nil]
L1:SMsgList --> nil
M:Msg --> pk(a, n(b, r1))
L2:SMsgList --> -(pk(b, n(a, r2))), +(n(a, r2) * n(b, r1)), nil
IK:IKnowledge --> nI(n(a, r2) * n(b, r1)), nI(pk(b, n(a, r2)))
state 32, State:
{[nil | +(pk(a, n(b, r1))), -(pk(b, n(a, r2))), +(n(a, r2) * n(b, r1)), nil] &
[nil | -(pk(a, n(b, r1))), +(pk(b, n(a, r2))), -(n(a, r2) * n(b, r1)), nil]
{nI(n(a, r2) * n(b, r1)), nI(pk(a, n(b, r1))), nI(pk(b, n(a, r2)))}}
accumulated substitution:
Y:Msg --> mt * n(a, r2)
X:Msg --> mt * n(b, r1)
Similarly to the search command, we can show the sequence of states and the label of each
transition, not showing the complete rule statement.
Maude> show path states 32 .
state 0, [State]:
{[nil, +(pk(a, n(b, r1))), -(pk(b, #1:Msg)), +(#1:Msg * n(b, r1)) | nil] &
[nil, -(pk(a, #2:Msg)), +(pk(b, n(a, r2))), -(#2:Msg * n(a, r2)) | nil]
{inI(#2:Msg * n(a, r2)), inI(pk(a, #2:Msg)), inI(pk(b, #1:Msg))}}
accumulated substitution:
Y:Msg --> #1:Msg
X:Msg --> #2:Msg
--- recv --->
state 1, State:
{[nil, -(pk(a, @2:XOR)), +(pk(b, n(a, r2))) | -(@2:XOR * n(a, r2)), nil] &
[nil, +(pk(a, n(b, r1))), -(pk(b, @1:XOR)), +(@1:XOR * n(b, r1)) | nil]
{inI(@2:XOR * n(a, r2)), inI(pk(a, @2:XOR)), inI(pk(b, @1:XOR))}}
accumulated substitution:
Y:Msg --> @1:XOR
X:Msg --> @2:XOR
--- snd --->
state 5, State:
{[nil, +(pk(a, n(b, r1))), -(pk(b, %1:XOR * n(a, r2))) | +(%1:XOR * n(a, r2) *
n(b, r1)), nil] &
[nil, -(pk(a, %1:XOR * n(b, r1))), +(pk(b, n(a, r2))) | -(%1:XOR * n(a, r2) *
n(b, r1)), nil]
{inI(pk(a, %1:XOR * n(b, r1))), inI(pk(b, %1:XOR * n(a, r2))), nI(%1:XOR *
n(a, r2) * n(b, r1))}}
accumulated substitution:
Y:Msg --> %1:XOR * n(a, r2)
X:Msg --> %1:XOR * n(b, r1)
--- recv --->
state 12, State:
{[nil, +(pk(a, n(b, r1))) | -(pk(b, @1:XOR * n(a, r2))), +(@1:XOR * n(a, r2) *
n(b, r1)), nil] &
[nil, -(pk(a, @1:XOR * n(b, r1))), +(pk(b, n(a, r2))) | -(@1:XOR * n(a, r2) *
n(b, r1)), nil]
{inI(pk(a, @1:XOR * n(b, r1))), inI(pk(b, @1:XOR * n(a, r2))), nI(@1:XOR *

<!-- manual page 421 -->
15.6. THE VU-NARROW COMMAND 409
n(a, r2) * n(b, r1))}}
accumulated substitution:
Y:Msg --> @1:XOR * n(a, r2)
X:Msg --> @1:XOR * n(b, r1)
--- snd --->
state 22, State:
{[nil, +(pk(a, n(b, r1))) | -(pk(b, n(a, r2))), +(n(a, r2) * n(b, r1)), nil] &
[nil, -(pk(a, n(b, r1))) | +(pk(b, n(a, r2))), -(n(a, r2) * n(b, r1)), nil]
{inI(pk(a, n(b, r1))), nI(n(a, r2) * n(b, r1)), nI(pk(b, n(a, r2)))}}
accumulated substitution:
Y:Msg --> mt * n(a, r2)
X:Msg --> mt * n(b, r1)
--- recv --->
state 28, State:
{[nil | -(pk(a, n(b, r1))), +(pk(b, n(a, r2))), -(n(a, r2) * n(b, r1)), nil] &
[nil, +(pk(a, n(b, r1))) | -(pk(b, n(a, r2))), +(n(a, r2) * n(b, r1)), nil]
{inI(pk(a, n(b, r1))), nI(n(a, r2) * n(b, r1)), nI(pk(b, n(a, r2)))}}
accumulated substitution:
Y:Msg --> mt * n(a, r2)
X:Msg --> mt * n(b, r1)
--- snd --->
state 32, State:
{[nil | +(pk(a, n(b, r1))), -(pk(b, n(a, r2))), +(n(a, r2) * n(b, r1)), nil] &
[nil | -(pk(a, n(b, r1))), +(pk(b, n(a, r2))), -(n(a, r2) * n(b, r1)), nil]
{nI(n(a, r2) * n(b, r1)), nI(pk(a, n(b, r1))), nI(pk(b, n(a, r2)))}}
accumulated substitution:
Y:Msg --> mt * n(a, r2)
X:Msg --> mt * n(b, r1)
15.6.6 Narrowing from multiple initial terms
The narrowing-based reachability command has been extended so that now an initial state can
be a disjunction of state terms instead of just a single stat term. Essentially, this provides us
with a forest of narrowing-based reachability trees instead of a single reachability tree. This
resembles an or-parallelism approach typical of logic programming (in the sense of Prolog) with
the difference that we assume the initial terms to have disjoint variables; this may be modified
in the future.
Given a system module ⟨ModId⟩, the user can give to Maude a narrowing-based search
command of the following form slightly different from the command in page 391:
{fold,path} vu-narrow {filter,delay} [ n, m ] in ⟨ModId⟩ :
⟨Term-1⟩ \/ \ldots \/ ⟨Term-n⟩ ⟨SearchArrow⟩ ⟨Term’⟩.
where the differences with the previous command are
• ⟨Term-1⟩,...,⟨Term-n⟩are the starting terms, which typically will contain variables but
they do not share any variable;
• ⟨Term’⟩is the pattern that has to be reached but, despite of the original command,
cannot share any variable with the starting terms.
For example, given the FOLDING-NARROWING-VENDING-MACHINE rewrite theory above, we
can ask whether it is possible to consume any two combinations of cookies and coffees in the
same command
{fold} vu-narrow in FOLDING-NARROWING-VENDING-MACHINE :

<!-- manual page 422 -->
410 CHAPTER 15. NARROWING
< M1:Marking a c > \/ < M2:Marking a a > \/ < M3:Marking c c > =>* < empty > .
Solution 1
rewrites: 9 in 1ms cpu (1ms real) (7240 rewrites/second)
state: < %1:Marking >
initial state: < M3:Marking c c >
accumulated substitution:
M3:Marking --> $ $ %1:Marking
variant unifier:
%1:Marking --> empty
Indeed, there is only one solution, since any of the three combinations requires two dollars.
Note that, when a disjunction of initial states is given, the information reported for each
reachability solution includes the initial state from which the state that unified with the pattern
was narrowed.
Note that, when the fold option is given, states from one tree may subsume states from
another tree. Indeed, one initial state may subsume another. For instance, we can see that
there is only one most general state and no open leaves.
Maude> show frontier states .
*** frontier is empty ***
Maude> show most general states .
< %1:Marking >
There is no equivalent command at the meta level for this feature. This may change in the
future.
15.7 Narrowing with extra variables in righthand sides of
rules
Althoughrewritingdoesnotallowextravariablesintherighthandsideofrules11, extravariables
in righthand sides pose no problem for narrowing. Since rules having extra variables in the
righthand side are not allowed in Maude for rewriting purposes, the attribute nonexec (see
Section 4.5.3) must be added to such rules if one wants to use them for narrowing. The
nonexec attribute is not taken into account by narrowing: all unconditional rules with the
narrowing attribute, regardless of whether or not they include the nonexec attribute, are used
by narrowing. Extra variables in the righthand side are a common feature of programs using
narrowing as the operational evaluation mechanism, as in logic programming or functional-logic
programming [70]. For further details on how to write funcional-logic programs in Maude using
symbolic reachability, see [55, 56]. Let us motivate this feature with an extended version of the
vending machine which is able to add as many coins as we want.
mod FOLDING-NARROWING-VENDING-MACHINE-EXTRAVAR is
sorts Coin Item Marking Money State .
subsort Coin < Money .
op empty : -> Money .
op __ : Money Money -> Money [assoc comm id: empty] .
subsort Money Item < Marking .
op __ : Marking Marking -> Marking [assoc comm id: empty] .
11Except for the case of conditional rules, where such extra variables may be introduced in matching or rewrite
conditions(seeChapter4); butconditionalruleshavebeenexcludedfromthepresentnarrowingimplementation,
as explained in Section 15.5.

<!-- manual page 423 -->
15.7. NARROWING WITH EXTRA VARIABLES IN RIGHTHAND SIDES OF RULES 411
op <_> : Marking -> State .
ops $ q : -> Coin .
ops a c : -> Item .
var M : Marking . var Mo : Money .
rl [buy-c] : < M $ c > => < M > [narrowing] .
rl [buy-a] : < M $ a > => < M q > [narrowing] .
rl [add] : < M > => < M Mo > [nonexec narrowing] .
eq [change] : q q q q M = $ M [variant] .
endm
This system module is clearly non-terminating by narrowing, since now the extra variable
Mo allows any additional coins.
Maude> vu-narrow [, 2] in FOLDING-NARROWING-VENDING-MACHINE-EXTRAVAR :
< empty > =>* < M > .
Solution 1
state: < empty >
accumulated substitution:
variant unifier:
M --> empty
Solution 2
state: < @1:Money >
accumulated substitution:
variant unifier:
M --> %1:Money
@1:Money --> %1:Money
Solution 3
state: < %1:Money %2:Money >
accumulated substitution:
variant unifier:
M --> @1:Money @2:Money
%1:Money --> @1:Money
%2:Money --> @2:Money
Solution 4
state: < %1:Money %2:Money >
accumulated substitution:
variant unifier:
M --> $ #1:Money #2:Money
%1:Money --> q q q #1:Money
%2:Money --> q #2:Money
Solution 5
state: < %1:Money %2:Money >
accumulated substitution:
variant unifier:
M --> $ #1:Money #2:Money
%1:Money --> q q #1:Money
%2:Money --> q q #2:Money
Solution 6
state: < %1:Money %2:Money >

<!-- manual page 424 -->
412 CHAPTER 15. NARROWING
accumulated substitution:
variant unifier:
M --> $ #1:Money #2:Money
%1:Money --> q #1:Money
%2:Money --> q q q #2:Money
Note that this symbolic reachability problem is not executable by rewriting, even if we start
from a ground state as in the previous command, but it’s executable by narrowing without any
problem.
Indeed, the symbolic reachability problem from the ground state has a finite reachability
graph computed by folding.
Maude> fvu-narrow in FOLDING-NARROWING-VENDING-MACHINE-EXTRAVAR :
< empty > =>* < M > .
Solution 1
state: < empty >
accumulated substitution:
variant unifier:
M --> empty
Solution 2
state: < @1:Money >
accumulated substitution:
variant unifier:
M --> %1:Money
@1:Money --> %1:Money

