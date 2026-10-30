# Chapter 4: Functional Modules (manual pages 55-96)

<!-- manual page 55 -->
Chapter 4
Functional Modules
Functional modules define data types and operations on them by means of equational theories.
The data types consist of elements that can be named by ground terms. Two ground terms
denote the same element if and only if they belong to the same equivalence class as determined
by the equations. That is, the mathematical semantics of a functional module is its initial
algebra. Maude’s functional modules are assumed to have the nice property that equations,
considered as simplification rules by using them only in the left to right direction, are Church-
Rosserandterminating(seeSection4.7). Thismeansthatrepeatedapplicationoftheequations
as simplification rules eventually reaches a term to which no further equations apply, and the
result, called the canonical form, is the same regardless of the order of application of the
equations. Thus each equivalence class has a natural representative, its canonical form, that
can be computed by equational simplification. As explained in Section 1.2, this ensures that
the initial algebra and the canonical term algebra of the functional module are isomorphic, and
therefore that the module’s mathematical and operational semantics coincide.
The equational logic on which Maude functional modules are based is an extension of order-
sorted equational logic [68] called membership equational logic [95, 14]. Thus, functional mod-
ules support multiple sorts, subsort relations, operator overloading, and assertions of member-
ship in a sort.
As was mentioned in Section 3.2, a functional module is declared in Maude using the key-
words
fmod ⟨ModuleName⟩ is ⟨DeclarationsAndStatements⟩ endfm
For example,
fmod NUMBERS is
...
endfm
declares a module named NUMBERS. The dots stand for the actual declarations and statements
that may appear in the functional module. Declarations include the importation of other
functional modules (see Chapter 7), and sort, subsort, and operator declarations. Statements
include equational and membership axioms. Declarations were discussed in Chapter 3. What
remains to be explained are equational and membership statements.
43

<!-- manual page 56 -->
44 CHAPTER 4. FUNCTIONAL MODULES
4.1 Unconditional equations
Unconditional equations are declared using the keyword eq, followed by a term (its lefthand
side), the equality sign=, then a term (its righthand side), optionally followed by a list of
statement attributes (see Section 4.5 later in this chapter) enclosed in square brackets, and
ending with white space and a period. Thus the general scheme is the following:
eq ⟨Term-1⟩= ⟨Term-2⟩ [⟨StatementAttributes⟩] .
The terms t and t’ in an equation t = t’ must both have the same kind. In order for the
equation to be executable, any variable appearing in t’ must also appear in t. Equations not
satisfying this requirement can also be declared (for example, to document a lemma holding
true in the module) but in such a case they should always be specified with the nonexec
attribute (see Section 4.5.3). We can add equations axiomatizing the addition operation in our
NUMBERS module as follows, where we distinguish two cases for the second argument, according
to whether it is zero or not:
vars N M : Nat .
eq N + zero = N .
eq N + s M = s (N + M) .
The following equations define the symmetric difference operation sd on natural numbers,
which returns the result of subtracting the smaller from the larger of its two arguments.
eq sd(N, N) = zero .
eq sd(N, zero) = N .
eq sd(zero, N) = N .
eq sd(s N, s M) = sd(N, M) .
In general, in a functional module one can specify equations (and also conditional equations,
as explained in Section 4.3) in three different ways:
1. in the style given above, in which case they are assumed to be executable as simplification
rules from left to right;
2. in the same style as above, but with the nonexec attribute (see Section 4.5.3), in which
case Maude does not use them for simplification (except at the metalevel with a user-given
strategy, see Section 17.7); and
3. as equational attributes of specific operators (see Section 4.4.1).
For example, a binary operator f can be declared assoc and comm, telling Maude that it
satisfies the associativity and commutativity axioms. Such equational attributes should not be
writtenexplicitlyasequationsinthespecification. Therearetworeasonsforthis. Firstly, thisis
redundant, since they have already been declared as equational attributes. Secondly, although
declaring such equations either only explicitly as equations, or twice—one time as equational
attributes and another as explicit equations—does not affect the mathematical semantics of the
specification, that is, the initial algebra that the specification denotes (see Section 4.3), it does
however drastically alter the specification’s operational semantics. For example, if the comm
attribute for f were to be stated as an equation f(X, Y) = f(Y, X), then using the equation
as a simplification rule applied to the term, say, f(a, b), would lead to the nonterminating
chain of equational simplifications
f(a, b) = f(b, a) = f(a, b) = f(b, a) = ...
This is quite bad, since we want the equations specified by method (1) to be used as
simplification rules and assume them to be terminating and Church-Rosser, so that they always
simplify a term to a unique result that cannot be further simplified. Instead, if comm is declared
as an equational attribute, the above kind of looping does not happen: Maude then simplifies

<!-- manual page 57 -->
4.2. UNCONDITIONAL MEMBERSHIPS 45
terms modulo the declared equational attributes, so that the terms f(a, b)and f(b, a)would
indeed be treated as identical. For more on equational attributes see Section 4.4.1.
4.2 Unconditional memberships
Unconditional membership axioms specify terms as having a given sort. They are declared with
the keyword mb followed by a term, followed by ‘:’, followed by a sort (that must always be
in the same kind as that of the term), followed by a period. As equations, memberships can
optionally have statement attributes (see Section 4.5).
mb ⟨Term⟩ : ⟨Sort⟩ [⟨StatementAttributes⟩] .
To illustrate this, consider the module 3*NAT with the basic Peano number declarations as
in the NUMBERS module and a new sort 3*Nat.
The fact that 3*Nat consists of multiples of 3 is expressed using the subsort declaration
Zero < 3*Nat < Nat and the membership statement mb (s s s M3) : 3*Nat for M3 a vari-
able of sort 3*Nat.
fmod 3*NAT is
sort Zero Nat .
subsort Zero < Nat .
op zero : -> Zero .
op s_ : Nat -> Nat .
sort 3*Nat .
subsorts Zero < 3*Nat < Nat .
var M3 : 3*Nat .
mb (s s s M3) : 3*Nat .
endfm
Memberships axioms can interact in undesirable ways with operators that are declared with
the assoc or iter attributes (see later Sections 4.4.1 and 4.4.2, respectively). This is explained
and illustrated with examples in Sections 21.3.8 and 21.3.9.
4.3 Conditional equations and memberships
Equational conditions in conditional equations and memberships are made up of individual
equations t= t′ and memberships t : s. A condition can be either a single equation, a single
membership, or a conjunction of equations and memberships using the binary conjunction
connective/\whichisassumedtobeassociative. Thusthegeneralformofconditionalequations
and memberships is the following:
ceq ⟨Term-1⟩= ⟨Term-2⟩
if ⟨EqCondition-1⟩ /\ ... /\ ⟨EqCondition-k⟩
[⟨StatementAttributes⟩] .
cmb ⟨Term⟩ : ⟨Sort⟩
if ⟨EqCondition-1⟩ /\ ... /\ ⟨EqCondition-k⟩
[⟨StatementAttributes⟩] .
Furthermore, the concrete syntax of equations in conditions has three variants, namely:
• ordinary equations t = t’,
• matching equations t := t’, and

<!-- manual page 58 -->
46 CHAPTER 4. FUNCTIONAL MODULES
• abbreviated Boolean equations of the form t, with t a term in the kind [Bool], abbrevi-
ating the equation t = true.
Any term t in the kind [Bool] can be used as an abbreviated Boolean1 equation. The
Boolean terms appearing most often in abbreviated Boolean equations are terms using the
built-in equality _==_ and inequality _=/=_ predicates, and the built-in membership predicates
_:: S with S a sort, including Boolean combinations of such terms with not_, _and_, _or_ and
other Boolean connectives (see Section 8.1 for a detailed description of all these operators). For
example, the following Boolean terms in the NUMBERS module (assuming that a “greater than”
operator _>_ has also been defined in NUMBERS),
N == zero
M =/= s zero
not (K :: NzNat)
(N > zero or M =/= s zero)
can appear as abbreviated Boolean equations in a condition, abbreviating, respectively, the
equations:
(N == zero) = true
(M =/= s zero) = true
not (K :: NzNat) = true
(N > zero or M =/= s zero) = true
To illustrate the use of conditional equations and memberships, let us reconsider the path
example from Section 3.5. The following conditional statements express the key membership
defining path concatenation and the associativity of this operator:
var E : Edge .
vars P Q R S : Path .
cmb E ; P : Path if target(E) = source(P) .
ceq (P ; Q) ; R = P ; (Q ; R)
if target(P) = source(Q) /\ target(Q) = source(R) .
The conditional membership axiom (introduced by the keyword cmb) states that an edge
concatenated with a path is also a path when the target node of the edge coincides with the
sourcenodeofthepath. Thishastheeffectofdefiningpathconcatenationasapartialoperation
on paths, although it is total on the kind [Path] of “confused paths.”
Assuming variables P, E, and S declared as above, source and target operations over paths
aredefinedbymeansofconditionalequationswithmatchingequationsinconditionsasfollows:2
ceq source(P) = source(E) if E ; S := P .
ceq target(P) = target(S) if E ; S := P .
Matching equations3 are mathematically interpreted as ordinary equations; however, opera-
tionally they are treated in a special way and they must satisfy special requirements. Note that
the variables E and S in the above matching equations do not appear in the lefthand sides of the
corresponding conditional equations. In the execution of these equations, these new variables
become instantiated by matching the term E ; Sagainst the canonical form of the subject term
bound to the variable P (see Section 4.7). In order for this match to decide the equality with
the ground term bound to P, the term E ; S must be a pattern. Given a functional module M,
1By default, any Maude module imports the predefined BOOL module (see Section 8.1).
2Note that the source and target operations can equivalently be declared as
eq source(E ; S) = source(E) .
eq target(E ; S) = target(S) .
3Similar constructs are used in languages like ASF+SDF [35] and ELAN [11].

<!-- manual page 59 -->
4.3. CONDITIONAL EQUATIONS AND MEMBERSHIPS 47
we call a term tan M-pattern if for any well-formed substitution σ such that for each variable
x in its domain the term σ(x) is in canonical form with respect to the equations in M, then
σ(t) is also in canonical form. A sufficient condition for tto be an M-pattern is the absence of
unifiers between its non-variable subterms and lefthand sides of equations in M.
Ordinary equations t= t′in conditions have the usual operational interpretation, that is, for
the given substitution σ, σ(t) and σ(t′) are both reduced to canonical form and are compared
for equality, modulo the equational attributes specified in the module’s operator declarations
such as associativity, commutativity, and identity. Finally, abbreviated Boolean equations are
just a special case of ordinary equations once they are expanded out.
The satisfaction of the conditions is attempted sequentially from left to right. Since in
Maude matching takes place modulo equational attributes, in general many different matches
may have to be tried until a match of all the variables satisfying the condition is found.
The above equations for source and target illustrate the use of matching equations to
bind variables locally, in much the same way that let is used in some functional programming
languages. In this example, since the matching is purely syntactic, the matching substitution is
unique and gives a simple way to name parts of a structure or to name a complicated expression
which appears multiple times in the main equation.
For M-patterns where some operators are matched modulo some equational attributes,
matching substitutions need not be unique. This provides another way of using matching
equations, namely to perform a search through a structure without any need to explicitly
define a function that does this. For example, for sequences of natural numbers we can define
a predicate _occurs-inner_ that determines if a number occurs in a sequence other than at
one of the ends. If one only cares about positive results,4 the following will work.
op _occurs-inner_ : [Nat] [NatSeq] -> [Bool] .
ceq N:Nat occurs-inner NS:NatSeq = true
if (NS0:NatSeq N:Nat NS1:NatSeq) := NS:NatSeq .
Note that this equation could also be written as
eq N:Nat occurs-inner NS0:NatSeq N:Nat NS1:NatSeq = true .
In both cases we check whether the sequence contains the natural number N:Nat, but making
sure that the sequence contains other elements both before and after N:Nat.
5 With the above
definition added to the numbers module, the term
zero occurs-inner (zero zero zero zero zero)
reduces to true, while the term
zero occurs-inner (zero zero)
does not reduce further.
Matching equations in conditions give great expressive power, but some care is needed in
using them to define operations. Consider adding the following to the numbers module, in an
attempt to define a test for the presence of s s zero in a sequence of natural numbers.
4Note that, since when the predicate is not true it remains unevaluated, we have defined it at the kind level,
that is, as a partial Boolean function; however, using the owise attribute (see Section 4.5.4) it is very easy to
add an extra equation making _occurs-inner_ a total Boolean function.
5Note that here we assume the declaration of the NatSeq concatenation operator__ as given in page 30,
where it is declared to be associative. If we consider the declaration of this operator given in page 49, which is
also declared to have nil as identity element, then we should write this equation as
op _occurs-inner_: [Nat] [NatSeq] -> [Bool] .
ceq N:Nat occurs-inner NS:NatSeq = true
if (I:Nat NS0:NatSeq N:Nat NS1:NatSeq M:Nat) := NS:NatSeq .
since the variables NS0:NatSeq and NS1:NatSeq might be instantiated to nil.

<!-- manual page 60 -->
48 CHAPTER 4. FUNCTIONAL MODULES
op hasTwo : [NatSeq] -> [Bool] .
ceq hasTwo(NS:NatSeq) = N:Nat == s s zero
if NS0:NatSeq N:Nat NS1:NatSeq := NS:NatSeq .
With this addition to the numbers module, hasTwo(zero zero) does not get reduced, since
the condition requires at least three numbers in the sequence. The term hasTwo(zero (s s
zero) zero) reduces to true. The term hasTwo(zero (s zero) (s s zero) zero) also gets
reduced, although it may return true or false; probably not what was intended. The problem
is that there are several matches, each giving a different answer, so the conditional equation
does not define a function. In fact, this conditional equation causes the Church-Rosser property
to fail, and semantically identifies true and false, thus leading to an inconsistent theory. In
contrast, as will be seen in Chapter 5, a rule with such a matching condition is not a problem,
and does have the effect of searching a sequence of natural numbers for s s zero.
In summary, all the sort, subsort, and operator declarations and all the statements in a
functional module (plus the functional modules imported if any) define an equational theory
in membership equational logic [95, 14]. Such a theory can be described in mathematical
notation as a pair (Σ,E∪A), where Σ is the signature, that is, the specification of the sorts,
subsorts, kinds, and operators in the module, E is the collection of statements (equations and
memberships, possibly conditional) and Ais the set of equational attributes, such as assoc and
comm, declared for some operators (that is, extra equations that are treated in a special way by
the Maude interpreter to simplify modulo such attributes, see Section 4.4.1).
The family of ground terms definable in the syntax of Σ defines a model called a Σ-algebra
and denoted TΣ. In TΣ, terms syntactically different denote different elements, so that TΣ will
not satisfy the equations in E∪A, unless they are trivial equations such as f(X) = f(X). The
question is, what is the optimal model of the theory (Σ,E∪A)? Goguen and Burstall’s answer
is: a model satisfying the axioms E∪Aand such that it has no junk (that is, all elements can
be denoted by ground Σ-terms), and no confusion (that is, only elements that are forced to
be equal by the axioms E∪A are identified). Such a model, called the initial algebra of the
equational theory (Σ,E∪A), exists [95], is denoted TΣ/E∪A, and provides the mathematical
semantics of the Maude functional module specifying (Σ,E∪A).
Mathematically, TΣ/E∪A can be constructed as the quotient of TΣ in which the equivalence
classes are those terms that are provably equal using the axioms E∪A. Operationally, assuming
that the axioms E are Church-Rosser and terminating modulo A (see Section 4.7), there is a
much more intuitive equivalent description of TΣ/E∪A, namely as the family of canonical forms
for the ground Σ-terms modulo A, that is, those terms that cannot be further simplified by the
equations in E modulo A. That is, as explained in Section 1.2, we have then an isomorphism
TΣ/E∪A
∼
= CanΣ/E∪A
between the initial algebra TΣ/E∪A and the canonical term algebra CanΣ/E∪A.
The Maude interpreter computes such canonical forms, which can be viewed as the val-
ues denoted by the corresponding functional expressions, with the reduce command (see Ap-
pendix A.2 for details and Section 4.9 for examples).
4.4 Operator attributes
Operator declarations may include attributes that provide additional information about the
operator: semantic, syntactic, pragmatic, etc. All such attributes are declared within a single
pair of enclosing square brackets, ‘[’ and ‘]’, after the sort of the result and before the ending
period. We discuss each of the categories of operator attributes below.

<!-- manual page 61 -->
4.4. OPERATOR ATTRIBUTES 49
4.4.1 Equational attributes
Equational attributes are a means of declaring certain kinds of equational axioms in a way that
allows Maude to use these equations efficiently in a built-in way. Currently Maude supports
the following equational attributes:
• assoc (associativity),
• comm (commutativity),
• idem (idempotency),
• id: ⟨Term⟩(identity, with the corresponding term for the identity element),
• left id: ⟨Term⟩(leftidentity, withthecorrespondingtermfortheleftidentityelement),
and
• right id: ⟨Term⟩(right identity, with the corresponding term for the right identity
element).
An operator can be declared with several of these attributes, which may appear in any order
in the attribute declaration. However, these attributes are only allowed for binary operators
satisfying the following requirements:
• For left id:, it is required that the right domain sort and the range sort belong to the
same kind.
• For right id:, it is required that the left domain sort and range sort belong to the same
kind.
• For assoc, comm, id:, and idem, both domain sorts and the range sort must belong to
the same kind.
These requirements are checked at parse time, and if the check fails a warning is output and
the operator loses its attributes.
Furthermore, we have the following additional requirements:
• The attribute idem cannot be used in any combination of attributes that includes assoc,
because the necessary matching and normalization algorithms have not been implemented
yet. This requirement is quietly enforced by ignoring the attribute idem where necessary.
• Only one identity attribute (left id:, right id:, or id:) is allowed. This is enforced
by a warning and by ignoring all but the first such attribute.
• Combining the attribute comm with either left id: or right id: silently turns the
identity attribute into an id:.
• All subsort-overloaded instances of an operator must have the same attributes. This is
further explained in Section 4.4.6.
Semantically, declaring a set of equational attributes for an operator is equivalent to declar-
ing the corresponding equations for the operator. Operationally, using equational attributes to
declare such equations avoids termination problems and leads to much more efficient evaluation
of terms containing such an operator. In fact, the effect of declaring equational attributes is to
compute with equivalence classes modulo such equations. This, besides being very expressive,
avoids what otherwise would be insoluble termination problems. For example, if a commutativ-
ity equation like x + y = y + x is declared as an ordinary equation, then it will easily produce
looping, nonterminating simplifications. If it is instead declared with an equational attribute
comm, this looping behavior does not happen.
In our numbers example we can add a constant nil for the empty sequence and refine the
declaration of sequence concatenation so that concatenation is associative with identity nil.
op nil : -> NatSeq .
op __ : NatSeq NatSeq -> NatSeq [assoc id: nil] .

<!-- manual page 62 -->
50 CHAPTER 4. FUNCTIONAL MODULES
As another example, we can form lists of Booleans as a supersort BList of Bool in an
extension of the BOOL module (see Section 8.1) with a “cons” operator_._ having nil as a right
identity:
sort BList .
subsort Bool < BList .
op nil : -> BList .
op _._ : Bool BList -> BList [right id: nil] .
Note that, when equational attributes are declared, equational simplification using the other
equations in the module does not take place at the purely syntactic level of replacing syntactic
equals by equals, but is understood modulo the equational attributes. Therefore, the proper
understanding of the notions of Church-Rosser and terminating equations, and of canonical
forms, is now modulo the equational attributes that have been declared. We discuss matching
and equational simplification modulo axioms in Section 4.8.
For example, by declaring the addition operation on natural numbers modulo 3 as commu-
tative,
op _+_ : Nat3 Nat3 -> Nat3 [comm] .
it is enough to have the following equations to define its behavior on all possible combinations
of arguments:
vars N3 : Nat3 .
eq N3 + 0 = N3 .
eq 1 + 1 = 2 .
eq 1 + 2 = 0 .
eq 2 + 2 = 1 .
The equations
eq 0 + N3 = N3 .
eq 2 + 1 = 0 .
are not needed, because they are subsumed by the first and third equations above, due to
commutativity of _+_.
Notice that membership axioms and matching modulo associativity can interact in undesir-
able ways, as explained in Section 21.3.8.
4.4.2 The iter attribute
Maude provides a built-in mechanism called the iter (short for iterated operator) theory whose
goal is to permit the efficient input, output, and manipulation of very large stacks of a unary
operator.
Unary operators may be declared to belong to the iter theory by including iter in their
attributes. After declaring
sort Foo .
op f : Foo -> Foo [iter] .
the term f(f(f(X:Foo))) can be input as f^3(X:Foo) and will be output in that form. A term
such as f^1234567890123456789(X:Foo) is too large to be input, output or manipulated in
regular notation, but can be input and output in this compact notation and certain (built-in)
manipulations may thus be made efficient.
The precise form of the compact iter theory notation is the prefix name of the operator
followed by ^[1-9][0-9]*(in Lex regular expression notation) with no intervening white space.

<!-- manual page 63 -->
4.4. OPERATOR ATTRIBUTES 51
Note that f^0123(X:Foo) is not acceptable. Of course, regular notation (and mixfix notation
if appropriate) can still be used.
Membership axioms may also interact in undesirable ways with operators declared with the
iter attribute; see Section 21.3.9 for details.
4.4.3 Constructors
Assuming that the equations in a functional module are (ground) Church-Rosser and termi-
nating, then every ground term in the module (that is, every term without variables) will be
simplified to a canonical form, perhaps modulo some declared equational attributes. Construc-
tors are the operators appearing in such canonical forms. The operators that “disappear” after
equational simplification are instead called defined functions. For example, typical construc-
tors in a sort Nat are zero and s_, whereas in the sort Bool, true and false are the only
constructors.
It is quite useful for different purposes, including both debugging (see Chapter 21) and
theorem proving, to specify when a given operator is a constructor. This can be done with
the ctor attribute. For example, we can refine our operator declarations in Section 3.4 with
constructor information as follows:
op zero : -> Zero [ctor] .
op s_ : Nat -> NzNat [ctor] .
op nil : -> NatSeq [ctor].
op __ : NatSeq NatSeq -> NatSeq [ctor assoc id: nil] .
Three slightly subtle points should be mentioned, namely the relationships of constructors
tooperatoroverloading, tokinds, andtoequations. Thefirstkeyobservationisthatconstructor
declarations are local to given sorts for the arguments and for the result. Nothing prevents an
operator from being a constructor at some level in the subsort ordering but being a defined
function at another. For example, we could have declared a successor function for integers,
op s_ : Int -> Int .
which is not a constructor. Indeed, we can define the sort Int with a subsort NzNeg of nonzero
negative numbers built up with a unary minus constructor-_, and we can then specify both
unary minus-_ and successor s_ as defined functions on the integers by giving the equations:
sorts NzNeg Int .
subsorts Nat NzNeg < Int .
op -_ : NzNat -> NzNeg [ctor] .
op -_ : Int -> Int .
op s_ : Int -> Int .
var N : Nat .
eq - zero = zero .
eq - (- (s N)) = s N .
eq s (- (s N)) = - N .
A related observation is that a defined function, which totally disappears at some level in
the subsort ordering, might not go away for terms at the kind level. For example, even though
addition may be a defined function, we may encounter an arithmetic error expression in a kind
of numbers such as
(s s zero) + p zero

<!-- manual page 64 -->
52 CHAPTER 4. FUNCTIONAL MODULES
because the predecessor function p has been declared on nonzero natural numbers.
op p : NzNat -> Nat .
The last point is that constructors may obey certain equations; that is, they do not have to
be free constructors. The equations that they may obey (even as constructors, not just in other
overloaded variants such as the integer successor function above) may be either equational
attributes (such as the assoc attribute in the above concatenation operator for strings of
natural numbers), or ordinary equations, or both. For example, we can add a sort NatSet of
finite sets of natural numbers to our NUMBERS module by declaring a set union operation _;_
using equational attributes to declare that it is associative and commutative with identity the
empty set, and using an ordinary equation to express idempotency.6
sort NatSet .
subsort Nat < NatSet .
op empty : -> NatSet [ctor] .
op _;_ : NatSet NatSet -> NatSet [ctor assoc comm id: empty] .
eq N ; N = N .
Given an equational specification in which several operators have been declared as construc-
tors by means of the ctor attribute and such that the equations are terminating, the sufficient
completeness problem consists in verifying that the canonical forms of all well-typed ground
terms are constructor terms. Intuitively, this means that all defined operations (i.e., those that
are not declared as constructors) have been fully defined. Maude’s Sufficient Completeness
Checker (SCC) can be used to ensure that constructor declarations are really correct, so that
all functions are fully defined relative to those constructors.7 We can take the NUMBERS module,
incrementally introduced in Chapter 3 and the previous sections of this chapter, to illustrate
how the SCC can be used to help the specifier in this regard.
fmod NUMBERS is
sort Zero .
sorts Nat NzNat .
subsort Zero NzNat < Nat .
op zero : -> Zero [ctor] .
op s_ : Nat -> NzNat [ctor] .
op sd : Nat Nat -> Nat .
ops _+_ _*_ : Nat Nat -> Nat .
op _+_ : NzNat Nat -> NzNat .
op p : NzNat -> Nat .
vars I N M : Nat .
eq N + zero = N .
eq N + s M = s (N + M) .
eq sd(N, N) = zero .
eq sd(N, zero) = N .
eq sd(zero, N) = N .
eq sd(s N, s M) = sd(N, M) .
sort Nat3 .
ops 0 1 2 : -> Nat3 .
6Remember that the idem attribute cannot be specified together with an assoc attribute; therefore idempo-
tency must in this case be specified explicitly by an equation.
7Information on how to use the SCC can be found at its web site (https://maude.cs.illinois.edu/tools/
scc/), throw MFE’s web site (https://github.com/maude-team/MFE/wiki), or throw Maude’s applications page
(https://maude.cs.illinois.edu/w/index.php/Maude_Tools).

<!-- manual page 65 -->
4.4. OPERATOR ATTRIBUTES 53
op _+_ : Nat3 Nat3 -> Nat3 [comm] .
vars N3 : Nat3 .
eq N3 + 0 = N3 .
eq 1 + 1 = 2 .
eq 1 + 2 = 0 .
eq 2 + 2 = 1 .
sort NatSeq .
subsort Nat < NatSeq .
op nil : -> NatSeq [ctor].
op __ : NatSeq NatSeq -> NatSeq [ctor assoc id: nil] .
sort NatSet .
subsort Nat < NatSet .
op empty : -> NatSet [ctor].
op _;_ : NatSet NatSet -> NatSet [ctor assoc comm id: empty] .
eq N ; N = N .
endfm
For expository reasons, since the ctor declaration had not yet been explained, some oper-
ators and constants were declared without the ctor attribute when they were introduced in
Section 3.6. The SCC reports the first term it finds not reducible to a constructor. In this case,
the first such report we get is the following:8
Maude> (scc NUMBERS .)
Checking sufficient completeness of NUMBERS ...
Warning: This module has equations that are not left-linear which
will be ignored when checking.
Failure: The term 0 was found to be a counterexample. Since the
analysis is incomplete, it may not be a real counterexample.
We fix this error by adding the ctor attribute to the declaration of the constants 0, 1, and 2
of sort Nat3:
ops 0 1 2 : -> Nat3 [ctor].
After this declaration is corrected, a more serious bug is found by the SCC, namely,
Maude> (scc NUMBERS .)
Checking sufficient completeness of NUMBERS ...
Warning: This module has equations that are not left-linear which
will be ignored when checking.
Failure: The term zero * zero was found to be a counterexample.
Since the analysis is incomplete, it may not be a real
counterexample.
Thismessageshowsthatthedefinitionofmultiplicationisincomplete, becausewehavedeclared
the operator without the ctor attribute but we have forgotten the equations defining such
operation on natural numbers. For example, we can add the following equations to make up
for this omission:
eq N * zero = zero .
eq N * s M = (N * M) + N .
8To execute these commands, the SCC tool must be running, and with the above NUMBERS module loaded.
Please, check the SCC’s documentation (https://maude.cs.illinois.edu/tools/scc/) for details.

<!-- manual page 66 -->
54 CHAPTER 4. FUNCTIONAL MODULES
A further iteration of the SCC on the amended specification shows that the equations for
the predecessor operation p are missing as well. Since p is only defined on nonzero natural
numbers, only one equation needs to be added:
eq p(s N) = N .
The corrected NUMBERS module after this analysis (together with some additional declara-
tions introduced in the following sections) is presented in Section 4.9. Here is the tool output
on the corrected module:
Maude> (scc NUMBERS .)
Checking sufficient completeness of NUMBERS ...
Warning: This module has equations that are not left-linear which
will be ignored when checking.
Success: NUMBERS is sufficiently complete under the assumption
that it is weakly-normalizing, confluent, and sort-decreasing.
4.4.4 Polymorphic operators
A number of Maude’s built-in operators are polymorphic in one or more arguments, in the
sense that the operator has meaning when these arguments are of any known sort. Examples
include Boolean operators such as the conditional, if_then_else_fi, which is polymorphic in
its second and third arguments, and the equality test _==_ which is polymorphic in both argu-
ments (see Section 8.1). The user can also define polymorphic operators using the polymorphic
attribute (abbreviated poly). This attribute takes a set of natural numbers enclosed in paren-
theses that indicates which arguments are polymorphic, with 0 indicating the range. For poly-
morphic operators that are not constants, at least one argument should be polymorphic to avoid
ambiguities. Since there are no polymorphic equations, polymorphic operators are limited to
constructors and built-ins. Polymorphic operators are always instantiated with the polymor-
phic arguments going to the kind level, which further limits their generality. The sort name
in a polymorphic position of an operator declaration is purely a place holder—any legal type
name could be used. The recommended convention is to use Universal.
One reasonable use for polymorphic operators beyond the existing built-ins is to define
heterogeneous lists, as follows, where CONVERSION denotes a predefined module described in
Section 8.10 having types for different numbers as well as strings; this module is imported by
means of a protecting declaration, which will be explained in Section 7.1.1.
fmod HET-LIST is
protecting CONVERSION .
sort List .
op nil : -> List .
op __ : Universal List -> List [ctor poly (1)] .
endfm
As an example, we can form the following heterogeneous lists:
Maude> red 4 "foo" 4.5 1/2 nil .
result List: 4 "foo" 4.5 1/2 nil
Maude> red (4 "foo" nil) 4.5 1/2 nil .
result List: (4 "foo" nil) 4.5 1/2 nil

<!-- manual page 67 -->
4.4. OPERATOR ATTRIBUTES 55
4.4.5 Format
The format attribute is intended to control the white space between tokens as well as color and
style when printing terms for programming-language-like specifications. Consider the following
mixfix syntax operator:
op (op_:_->_[_].) : Qid TypeList Type AttrSet -> OpDecl .
There are eleven places where white space can be inserted:
op _ : _ -> _ [ _ ] .
^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^
A format attribute must have an instruction word for each of these places. For example,
the formatting specification for the above operator could be chosen to be:
[format (d d d d d d s d d s d)]
Instruction words are formed from the following alphabet:
d default spacing
(cannot be part of a larger word: must occur on its own)
+ increment global indent counter
- decrement global indent counter
s space
t tab
i number of spaces determined by indent counter
n newline
Note that, in general, each place may have an entire word combining several of the above
symbols. We can illustrate how this feature is used in several operators in (submodules of) the
META-LEVEL module in the file prelude.maude (see Chapter 17).
• Each assignment will be printed in a new line, indented one tab.
op _<-_ : Variable Term -> Assignment
[ctor prec 63 format (nt d d d)] .
• Each importation after the first one will be printed in a new line, with the current inden-
tation.
op __ : ImportList ImportList -> ImportList
[ctor assoc id: nil format (d ni d)] .
• Each kind of declaration in a module will start in a new line, with the current indentation,
which is increased by two at the beginning and decreased by two at the end of the module.
op fmod_is_sorts_.____endfm : Qid ImportList SortSet
SubsortDeclSet OpDeclSet MembAxSet EquationSet -> FModule
[ctor gather (& & & & & & &)
format (d d d n++i ni d d ni ni ni ni n--i d)] .
Whether the format attribute is actually used or not when printing is controlled by the
command:
set print format on/off .
The following additional alphabet can be used to change the text color and style. These
colors, perhaps combined with spacing directives, can greatly ease readability, particularly in
complex terms for which they can serve as markers. They rely on ANSI escape sequences which

<!-- manual page 68 -->
56 CHAPTER 4. FUNCTIONAL MODULES
are supported by most terminal emulators, most notably the Linux console, Xterm, and Mac
Terminal windows, but not Emacs shell buffers, unless you use ansi-color.el.
9
r red
g green
y yellow
b blue
m magenta
c cyan
u underline
! bold
o revert to original color and style
By default ANSI escape sequences are suppressed if the environment variable TERM is set
equal to dumb (Emacs does this) or standard output is not a terminal; they are allowed oth-
erwise. This behavior can be overridden by the command line options -ansi-color and
-no-ansi-color.
You are allowed to give a format attribute even if there is no mixfix syntax. In this case
the format attribute must have two instruction words, indicating the desired format before and
after the operator’s name. For example,
fmod COLOR-TEST is
sorts Color ColorList .
subsort Color < ColorList .
op red : -> Color [format (r! o)] .
op green : -> Color [format (g! o)] .
op blue : -> Color [format (b! o)] .
op yellow : -> Color [format (yu o)] .
op cyan : -> Color [format (cu o)] .
op magenta : -> Color [format (mu o)] .
op __ : ColorList ColorList -> ColorList [assoc] .
endfm
To see the colors in this module, load the COLOR-TEST module into Maude and execute the
command:10
Maude> reduce red green blue yellow cyan magenta .
reduce in COLOR-TEST : red green blue yellow cyan magenta .
rewrites: 0 in 0ms cpu (0ms real) (~ rews/sec)
result ColorList: red green blue yellow cyan magenta
Let us consider the following module FORMAT-DEMO, where a small programming language
is defined.
fmod FORMAT-DEMO is
sorts Variable Expression BoolExp Statement .
subsort Variable < Expression .
ops a b c d : -> Variable .
op 1 : -> Expression .
op _+_ : Expression Expression -> Expression [assoc comm] .
op _;_ : Statement Statement -> Statement [assoc prec 50] .
op _<=_ : Expression Expression -> BoolExp .
9There is a copy of this Emacs Lisp file with the Maude distribution just in case your Emacs distribution
lacks it.
10Try it in your terminal. The colors are not shown here for obvious reasons.

<!-- manual page 69 -->
4.4. OPERATOR ATTRIBUTES 57
op while_do_od : BoolExp Statement -> Statement
[format (nir! o r! o++ --nir! o)] .
op let_:=_ : Variable Expression -> Statement
[format (nir! o d d d)] .
endfm
Note the use of the format attribute for operators while_do_od and let_:=_. Since both
represent statements, which should start in a new line, but at the current indentation level,
bothincludeniintheinstructionwordsfortheirfirstpositions; thispositionalsohascharacters
r! in both cases, so that they start in boldface red font. Since there is a o for the next position,
reverting to original color and style, only the first word (while and let) is shown in red. In
the case of while_do_od, the condition of the loop starts at the second position. The do word
is shown in boldface red, and then the indentation counter is incremented, so that the body
of the while_do_od statement is indented. For the position marking the beginning of od, the
counter is decremented, so that it appears at the level of while in a new line (n), in boldface
red font (r!). The last position reverts the original color and style, although notice that the
indentation counter remains the same, so that successive statements will be given the same
level of indentation. In the case of let_:=_, the three last positions contain only d (default
spacing), since it is to be presented as a single-line statement in which let is shown in boldface
red.
We can illustrate the difference between using the format attribute and not using it with
the following commands (as before, you should execute the example in your terminal to see the
colors).
Maude> set print format off .
Maude> parse
while a <= d do
let a := a + b ;
while b <= d do
let b := b + c ;
let c := c + 1
od
od
.
Statement: while a <= d do let a := a + b ; while b <= d do let b :=
b + c ; let c := c + 1 od od
Maude> set print format on .
Maude> parse
while a <= d do
let a := a + b ;
while b <= d do
let b := b + c ;
let c := c + 1
od
od
.
Statement:
while a <= d do
let a := a + b ;
while b <= d do
let b := b + c ;

<!-- manual page 70 -->
58 CHAPTER 4. FUNCTIONAL MODULES
let c := c + 1
od
od
For more examples of format attributes, you can see the operator declarations in the module
LTL (in the file model-checker.maude) discussed in Chapter 12, or in the modules META-TERM
and META-MODULE (in the file prelude.maude), described in Chapter 17.
4.4.6 Ditto
An operator can have several subsort-overloaded instances. Maude requires that all these
instances should have the same attributes, except for the case of the ctor attribute, that may
be present in some instances but absent in others (see Section 4.4.3), and/or the metadata
attribute (see Section 4.5.2). It is for example forbidden to have a subsort-overloaded instance
in which an operator is declared assoc only, and another such instance in which it is declared
assoc and comm.
The ditto attribute can be given to an operator for which another subsort-overloaded
instancehas already appeared,eitherinthesamemoduleorinasubmodule. Thedittoattribute
is just a shorthand stating that this operator, being subsort overloaded, should have the same
attributes as those appearing explicitly in a previous subsort-overloaded version, except for the
ctor and metadata attributes, which are outside the scope of ditto. In this way we can avoid
writing out a possibly long attribute list again and again.
It is not allowed to combine ditto with other attributes, except for ctor and metadata.
That is, an operator given the ditto attribute either has no other explicitly given attributes,
or can only have in addition either the ctor attribute if it is a constructor, or a metadata
attribute, or both the ctor and metadata attributes. Furthermore, it is forbidden to use ditto
on the first declared instance of an operator, since this is nonsensical.
In our numbers module we can add equational attributes to the declarations of _+_ and
_*_, and then use ditto to declare the same attributes in other subsort-overloaded versions.
ops _+_ _*_ : Nat Nat -> Nat [assoc comm].
op _+_ : NzNat Nat -> NzNat [ditto] .
op _*_ : NzNat NzNat -> NzNat [ditto] .
ForanexamplemakingextensiveuseofthedittoattributeseetheLTL-SIMPLIFIERmodule
(in the file model-checker.maude), discussed in Chapter 12.
4.4.7 Operator evaluation strategies
If a collection of equations is Church-Rosser and terminating, given an expression, no matter
how the equations are used from left to right as simplification rules, we will always reach the
same final result. However, even though the final result may be the same, some orders of
evaluation can be considerably more efficient than others. More generally, we may be able to
achieve the termination property provided we follow a certain order of evaluation, but may lose
termination when any evaluation order is allowed. It may therefore be useful to have some way
of controlling the way in which equations are applied by means of strategies.
In general, given an expression f(t1,...,tn) we can try to evaluate it to its reduced form in
different ways, such as:
• first obtaining the reduced form of all the ti and then applying equations for f at the top
of the term; this is called a bottom-up, or eager strategy;
• evaluating only some of the arguments, and then trying to evaluate at the top with
equations for f; for example, an if_then_else_fi operator will typically be evaluated

<!-- manual page 71 -->
4.4. OPERATOR ATTRIBUTES 59
by evaluating first the first argument, and then the if_then_else_fi operator at the
top;
• trying to evaluate the top of the term first, and then, if this fails, either not evaluating
the subterms at all, or trying to evaluate only some of them, that is, some kind of lazy
evaluation strategy.
Typically, a functional language is either eager, or lazy with some strictness analysis added
for efficiency, and the user has to live with whatever the language provides. Maude adopts
OBJ3’s [69] flexible method of user-specified evaluation strategies on an operator-by-operator
basis, addingsomeimprovementstotheOBJ3approachtoensureacorrectimplementation[50].
For an n-ary operator f an evaluation strategy is specified as a list of numbers from 0 to n
ending with 0. The nonzero numbers denote argument positions, and a 0 indicates evaluation
at the top of the given function symbol. The strategy then specifies what argument positions
mustbesimplified(intheorderindicatedbythelist)beforeattemptingsimplificationatthetop
with the equations for the top function symbol. In functional programming terminology, the
argument positions to be evaluated are usually called strict argument positions, so we can view
an evaluation strategy as a flexible, user-definable way of specifying strictness requirements
on argument positions. In the simplest case, a strategy consists of a list of nonzero numbers
followed by a 0, so that some arguments are treated strictly and then the function symbol’s
equations are applied. For example, in Maude, if no strategy is specified, all argument positions
are assumed strict, so that for f with n argument positions its default strategy is (1 2 ...n0);
this is the “eager evaluation” case. The opposite extreme is a form of lazy evaluation such as
the lazy append operator in the SIEVE example below. This operator has strategy (0), thus
only equations at the top are tried during evaluation.
The syntax to declare an n-ary operator with strategy (i1 ... ik 0), where ij ∈{0,...,n}
for j = 1,...,k, is
op ⟨OpName⟩ : ⟨Sort-1⟩... ⟨Sort-n⟩ -> ⟨Sort⟩ [strat (i1... ik 0)] .
As a simple example consider the operators _and-then_ and _or-else_ in the module
EXT-BOOL, that can be found in the file prelude.maude (see Section 8.1).
fmod EXT-BOOL is
protecting BOOL .
op _and-then_ : Bool Bool -> Bool
[strat (1 0) gather (e E) prec 55] .
op _or-else_ : Bool Bool -> Bool
[strat (1 0) gather (e E) prec 59] .
var B : [Bool] .
eq true and-then B = B .
eq false and-then B = false .
eq true or-else B = true .
eq false or-else B = B .
endfm
These operators are computationally more efficient versions of Boolean conjunction and
disjunction that avoid evaluating the second of the two Boolean subterms in their arguments
when the result of evaluating the first subterm provides enough information to compute the
conjunction or the disjunction. For example, letting B:[Bool] stand for an arbitrary Boolean
expression
Maude> red false and-then B:[Bool] .
result Bool: false

<!-- manual page 72 -->
60 CHAPTER 4. FUNCTIONAL MODULES
while if B:[Bool] does not evaluate to true or false, then false and B:[Bool] does not
evaluatetofalse, andifevaluationof B:[Bool]doesnotterminatethenneitherwillevaluation
of false and B:[Bool].
If some of the argument positions are never mentioned in some of the operator strategies,
the notion of canonical form becomes now relative to the given strategies and may not coincide
with the standard notion. Let us consider as a simple example the following two functional
modules, which we have displayed side-by-side to emphasize their only difference, namely, the
evaluation strategy associated to the operator g.
fmod STRAT-EX1 is sort S . ops a b : -> S . op g : S -> S . eq a = b . endfm endfm
fmod STRAT-EX2 is
sort S .
ops a b : -> S .
op g : S -> S [strat(0)] .
eq a = b .
The canonical form of the term g(a) in STRAT-EX1 is g(b), but in STRAT-EX2 it is g(a)
itself, because the equation cannot be applied inside the term due to the lazy strategy strat(0)
of the operator g.
This may be just what we want, since we may be able to achieve termination to a canonical
form relative to some strategies in cases when the equations may be nonterminating in the
standard sense. More generally, operator strategies may allow us to compute with infinite data
structures which are evaluated on demand, such as the following formulation of the sieve of
Eratosthenes, which finds all prime numbers using lazy lists.
The infinite list of primes is obtained from the infinite list of all natural numbers greater
than 1 by filtering out all the multiples of numbers previously taken. Thus, first we take 2
and delete all even numbers greater than 2; then we take 3 and delete all the multiples of 3
greater than 3; and so on. The operation nats-from_ generates the infinite list of natural
numbers starting in the given argument; the operation filter_with_ is used to delete all the
multiples of the number given as second argument in the list provided as first argument; and
the operation sieve_ is used to iterate this process with successive numbers.
Of course, since we are working with infinite lists, we cannot obtain as a result an infinite
list. Such an infinite structure is only shown partially by means of the operation show_upto_,
which shows only a finite prefix of the whole infinite list. Moreover, the generation and filtering
processes have to be done in a lazy way. This is accomplished by giving to the list constructor
_._ a lazy strategy strat(0) that avoids evaluating inside the term, and using an operation
force with an eager strategy strat(1 2 0) to “force” the evaluation of elements inside the
list. Specifically, in order to apply the first equation, we must evaluate the arguments L and S
before reconstructing the list L . S in the righthand side.
NAT denotes the predefined module of natural numbers and arithmetic operations on them
(see Section 8.3), which is imported by means of a protecting declaration, explained in Sec-
tion 7.1.1. Note the use of the symmetric difference operator sd (see Section 8.3) to decrement
I in the third equation, and the successor operator s_ to increment I in the sixth equation.
fmod SIEVE is
protecting NAT .
sort NatList .
subsort Nat < NatList .
op nil : -> NatList .
op _._ : NatList NatList -> NatList [assoc id: nil strat (0)] .
op force : NatList NatList -> NatList [strat (1 2 0)] .
op show_upto_ : NatList Nat -> NatList .

<!-- manual page 73 -->
4.4. OPERATOR ATTRIBUTES 61
op filter_with_ : NatList Nat -> NatList .
op nats-from_ : Nat -> NatList .
op sieve_ : NatList -> NatList .
op primes : -> NatList .
vars P I E : Nat .
vars S L : NatList .
eq force(L, S) = L . S .
eq show nil upto I = nil .
eq show E . S upto I
= if I == 0
then nil
else force(E, show S upto sd(I, 1))
fi .
eq filter nil with P = nil .
eq filter I . S with P
= if (I rem P) == 0
then filter S with P
else I . filter S with P
fi .
eq nats-from I = I . nats-from (s I) .
eq sieve nil = nil .
eq sieve (I . S) = I . sieve (filter S with I) .
eq primes = sieve nats-from 2 .
endfm
We can then evaluate expressions in this module with the reducecommand (see Sections 4.9
and Appendix A.2). For example, to compute the list of the first ten prime numbers we evaluate
the expression:
Maude> reduce show primes upto 10 .
result NatList: 2 . 3 . 5 . 7 . 11 . 13 . 17 . 19 . 23 . 29
In the case of associative or commutative binary operators, evaluation strategies might
reduce some arguments that the user does not expect to be reduced. The reason is that in such
cases terms represent equivalence classes and it might be quite hard to say what is the first or
the second argument. The adopted solution is that mentioning either argument implies both.
The paper [50] documents the operational semantics and the implementation techniques for
Maude’s operator evaluation strategies in much more detail. The mathematical semantics of
a Maude functional module having operator evaluation strategies is documented in [71] and is
further discussed in Section 4.7.
Of course, operator evaluation strategies, while quite useful, are by design restricted in their
scope of applicability to functional modules.
11 As we shall see in Chapter 5, system modules,
specifying rewrite theories that are not functional, need not be Church-Rosser or terminating,
and require much more general notions of strategy. Such general strategies are provided by
Maude using the strategy language documented in Section 10, in which strategies are defined
as high-level programs where rule application is the basic instruction, and also using reflection
by means of internal strategies (see Section 17.7). However, as discussed in Section 4.4.9,
specifying frozen arguments in operators restricts the rewrites allowed with rules in a system
11More precisely, the scope of applicability of operator evaluation strategies is restricted to functional modules
and to the equational part of system modules.

<!-- manual page 74 -->
62 CHAPTER 4. FUNCTIONAL MODULES
module (as opposed to equations) in a way quite similar to how operator evaluation strategies
restrict the application of equations in a functional module.
4.4.8 Memo
If an operator is given the memo attribute, this instructs Maude to memoize the results of
equational simplification (that is, the canonical forms) for those subterms having that operator
at the top. This means that when the canonical form of a subterm having that operator at
the top is obtained, an entry associating to that subterm its canonical form is stored in the
memoization table for this operator. Whenever the Maude interpreter encounters a subterm
whose top operator has the memo attribute, it looks to see if its canonical form is already
stored. If so, that result is used; otherwise, equational simplification proceeds according to
the operator’s strategy. Giving to some operators the memo attribute allows trading off space
for time in equational simplifications: more space is needed, but if subcomputations involving
memoized operators have to be repeated many times, then a computation may be substantially
sped up, provided that the machine’s main memory limits are not exceeded.
An operator’s memo attribute and its user’s specified or default evaluation strategy (see
Section 4.4.7) may interact with each other, impacting the size of the memoization table. The
issue is how many entries for different subterms, all having the same canonical form, may be
possibly stored in the memoization table. If the operator has the default, bottom-up strategy,
the answer is: only one such entry is possible. For other strategies, different terms having the
same canonical form may be stored, making the memoization table bigger. For example, using
the default strategy (1 2 0) for a memoized operator f, only subterms of the form f(v, v’)
with v and v’ fully reduced to canonical form (up to the strategies given for all operators) will
be mapped to their corresponding canonical forms. This is because, with the default strategy,
equational simplification at the top of f can only happen after all its arguments are in canonical
form. For other operator strategies this uniqueness may be lost, even when evaluating just one
subterm involving f. For example, if f’s strategy is (0 1 2 0), then both the starting term
f(t, t’) and the term f(v, v’) (where v and v’ are, respectively, the canonical forms of
t and t’) will be mapped to the final result, since the strategy specifies rewriting at the top
twice. That is, each time the operator’s strategy calls for rewriting at the top, Maude will
add the current version of the term to the set of terms that will be mapped to the final result.
Furthermore, other terms of the form f(u, u’), with u and u’ having also v and v’ as their
canonical forms may appear in other subcomputations, and will then also be stored in the
memoization table.
In general, whenever an application will perform an operation many times, it may be useful
to give that operator the memo attribute. This may be due to the high frequency with which the
operator is called by other operators in a given application, or to the highly recursive nature
of the equations defining that operator. For example, the recursive definition of the Fibonacci
function is given as follows, where NAT denotes the predefined module of natural numbers and
arithmetic operations on them (as described in Section 8.3), which is imported by means of a
protecting declaration (see Section 7.1.1).
fmod FIBONACCI is
protecting NAT .
op fibo : Nat -> Nat .
var N : Nat .
eq fibo(0) = 0 .
eq fibo(1) = 1 .
eq fibo(s s N) = fibo(N) + fibo(s N) .

<!-- manual page 75 -->
4.4. OPERATOR ATTRIBUTES 63
endfm
Due to the highly recursive nature of this definition of fibo, the evaluation of an expression
like fibo(50) will compute many calls to the same instances of the function again and again,
and will expand the original term into a whole binary tree of additions before collapsing it to a
number. The exponential number of repeated function calls makes the evaluation of fibo with
the above equations very inefficient, requiring over 61 billion rewrite steps for fibo(50):
Maude> red fibo(50) .
reduce in FIBONACCI : fibo(50) .
rewrites: 61095033220 in 132081000ms cpu (145961720ms real)
(462557 rews/sec)
result NzNat: 12586269025
If we instead give the Fibonacci function the memo attribute,
op fibo : Nat -> Nat [memo] .
the change in performance is quite dramatic:
Maude> red fibo(50) .
reduce in FIBONACCI : fibo(50) .
rewrites: 148 in 0ms cpu (0ms real) (~ rews/sec)
result NzNat: 12586269025
Maude> red fibo(100) .
reduce in FIBONACCI : fibo(100) .
rewrites: 151 in 0ms cpu (1ms real) (~ rews/sec)
result NzNat: 354224848179261915075
Maude> red fibo(1000) .
reduce in FIBONACCI : fibo(1000) .
rewrites: 2701 in 0ms cpu (11ms real) (~ rews/sec)
result NzNat: 434665576869374564356885276750406258025646605173717804
024817290895365554179490518904038798400792551692959225930803226347
752096896232398733224711616429964409065331879382989696499285160037
04476137795166849228875
In some cases we may introduce a constant operator as an abbreviation for a possibly
complex expression that may require a substantial number of equational simplification steps to
be reduced to canonical form; furthermore, the operator may be used repeatedly in different
subcomputations. In such cases one can declare a constant operator, give it the memo attribute,
and give an equation defining it to be equal to the expression of interest. For example, suppose
we have defined a search space with initial state myState and a function findAnswer to search
the space for a state satisfying some property. Then we can name the search result and use it
again without redoing an expensive computation as follows:
op myAns : -> Answer [memo] .
eq myAns = findAnswer(myState) .
Maudewillthenremembertheresultofrewritingtheconstantinthememoizationtableforthat
operator and will not repeat the work until the memoization tables are cleared. Memoization
tables for the current module can be cleared explicitly by the command
do clear memo .
Automatic clearing before each top level rewriting command can be turned on and off with

<!-- manual page 76 -->
64 CHAPTER 4. FUNCTIONAL MODULES
set clear memo on .
set clear memo off .
By default, set clear memo is off.
4.4.9 Frozen arguments
The frozen attribute is only meaningful for system modules (see Chapter 5) that may have
both rules and equations. It has no direct effect for functional modules having only equations
and memberships: it can only have an indirect effect if the functional module is later imported
by a system module. For this reason, examples of the use of frozen operators are postponed to
Chapter 5.
Given a system module M, by declaring a given operator, say f, as frozen, rewriting with
rules is always forbidden in all proper subterms of a term having fas its top operator. However,
itmaystillbepossibletorewritethattermatthetop, providedruleshaving fasthetopsymbol
of their lefthand side exist in M. To specify that all the arguments of an operator are frozen,
one includes the attribute frozen in the operator’s list of attributes; for example,
op f : S1 ... Sn -> S [frozen] .
The freezing idea can be generalized, so that only specific argument positions of the operator
f are frozen. For example, in a system module specifying the semantics of a programming
language with rewrite rules, we may want to specify a sequential composition operator _;_ as
frozen in its second argument, but not in the first argument, so as to prevent any execution of
the second program fragment of the composition from happening before the first fragment has
been fully evaluated. We can specify this by stating
op _;_ : Program Program -> Program [frozen (2)] .
More generally, if the list of argument positions in an operator f is 1 ...n, then we can
freeze any sublist of argument positions, say i1 ...im, by declaring,
op f : S1 ... Sn -> S [frozen (i1 ... im)] .
Of course, if the actual list of specified positions is 1 ...n itself, then this is equivalent to
the first mode of declaring the frozen attribute for f without listing any positions.
As for operator evaluation strategies (see Section 4.4.7), in the case of associative or com-
mutative binary operators mentioning either argument in the list of frozen positions implies
both.
4.4.10 Special
Many operators in predefined modules (see Chapters 8 and 17) have the special attribute
in their declarations. This means that they are to be treated as built-in operators, so that,
instead of having the standard treatment of any user-defined operator, they are associated with
appropriate C++ code by “hooks” which are specified following the specialattribute identifier.
For example, the file prelude.maude contains a predefined module NAT for natural numbers
and usual operations on them (see Section 8.3). Among others, the declarations in the NAT
module for the operations of addition and of quotient of integer division, and for a less than
predicate are the following:
op _+_ : NzNat Nat -> NzNat
[assoc comm prec 33
special (id-hook ACU_NumberOpSymbol (+)
op-hook succSymbol (s_ : Nat ~> NzNat))] .

<!-- manual page 77 -->
4.5. STATEMENT ATTRIBUTES 65
op _+_ : Nat Nat -> Nat [ditto] .
op _quo_ : Nat NzNat -> Nat
[prec 31 gather (E e)
special (id-hook NumberOpSymbol (quo)
op-hook succSymbol (s_ : Nat ~> NzNat))] .
op _<_ : Nat Nat -> Bool
[prec 37
special (id-hook NumberOpSymbol (<)
op-hook succSymbol (s_ : Nat ~> NzNat)
term-hook trueTerm (true)
term-hook falseTerm (false))] .
Notice that the special attribute exists in order to bind Maude syntax to built-in C++
functionality. It is absolutely not for users to mess with and it is absolutely not backwards
compatible; this is why Maude will sometimes crash or become unstable if the prelude from a
different version is loaded. For the same reason, other operator attributes that appear together
with special in an operator declaration cannot be modified either.
4.5 Statement attributes
In a functional module, statements are equations and membership axioms, conditional or not.
Any such statement may have associated attributes. Currently five attributes are available:
label, metadata, nonexec, owise, and print. The attributes label, metadata, nonexec, and
print can also be used on rules in system modules. Moreover, the attribute metadata can also
be associated to operator declarations. The dnt attribute is only available for statements in
object-oriented modules. It is discussed in Section 6.1.4.
4.5.1 Labels
The label attribute must be followed by an identifier. Statement labels can be used for tracing
and debugging and at the metalevel to name particular axioms. In our numbers example
(Section 4.4.3) we could label the axiom for idempotency for natural number sets
eq N ; N = N [label natset-idem] .
Syntactic sugar for labels generalizing the Maude 1 style for rule labels is also supported.
Then the above label could have also been written
eq [natset-idem] : N ; N = N .
4.5.2 Metadata
The metadata attribute must be followed by a string (that is, by a data element in the STRING
module, see Section 8.9). The metadata attribute is intended to hold data about the statement
in whatever syntax the user cares to create/parse. It is like a comment that is carried around
with the statement. Usual string escape conventions apply. For example, we could add the
distributive law
eq (N + M) * I = (N * I) + (M * I) [metadata "distributive law"] .

<!-- manual page 78 -->
66 CHAPTER 4. FUNCTIONAL MODULES
with the comment documenting that this is the distributive law.
The metadata attribute can also be associated to operator declarations. Note that, like
ctor, metadata is attached to a specific operator declaration and not to the (possibly over-
loaded) operator itself. Thus:
• two subsorted overloaded declarations may have different metadata attributes,
• a metadata attribute is not copied by the ditto attribute (see Section 4.4.6), and
• a declaration may have a metadata attribute as well as a ditto attribute.
Under these conditions, the following ad-hoc example is therefore legal:
fmod METADATA-EX is
sorts Foo Bar .
subsort Foo < Bar .
op f : Foo -> Foo [memo metadata "f on Foos"] .
op f : Bar -> Bar [ditto metadata "f on Bars"] .
endfm
4.5.3 Nonexec
The nonexec attribute allows the user to include statements in a module that are ignored by
the Maude rewrite engine. For example we could make the distributive law nonexecutable as
follows.
eq (N + M) * I = (N * I) + (M * I)
[nonexec metadata "distributive law"] .
Similarly, a rule can be declared with the nonexec attribute in a system module.
Although nonexecutable from the point of view of Maude, such statements are part of the
semantics of the module and can for example be used at the metalevel for controlled execution
or theorem proving purposes.
4.5.4 Otherwise
Sometimes, in the definition of an operation by equations, there are certain cases that can be
easily defined by equations, and then some remaining case or cases that it is more difficult
or cumbersome to define. One would in such situations like to say, otherwise, that is, in all
remaining cases not covered by the above equations, do so and so.12
Consider, for example, the problem of membership of a natural number in a finite set of
numbers.
op _in_ : Nat NatSet -> Bool .
The easy part is to define when a number belongs to a set:
var N : Nat .
var NS : NatSet .
eq N in N ; NS = true .
It is somewhat more involved to define when it does not belong. A simple way is to use the
otherwise (abbreviated owise) attribute and give the additional equation:
eq N in NS = false [owise] .
12Indeed, several languages have conventions of this kind, including ASF+SDF [35].

<!-- manual page 79 -->
4.5. STATEMENT ATTRIBUTES 67
The intuitive operational meaning is clear: if the first equation does not match, then the
number in fact is not in the set, and the predicate should be false. But what is the mathematical
meaning? That is, how can we interpret the meaning of the second equation so that it becomes
a useful shorthand for an ordinary equation? After all, the second equation, as given, is even
more general than the first and in direct contradiction with it. We of course should reject any
constructs that violate the logical semantics of the language.
Fortunately, there is nothing to worry about, since the owiseattribute is indeed a shorthand
foracorrespondingconditional equation. Wefirstexplaintheideainthecontextofthisexample
and then discuss the general construction. The idea is that, whether an equation, or a set of
equations, definingthemeaningofanoperationf matchagiventerm, isitselfapropertydefined
by a predicate, say enabledf, which is effectively definable by equations. In our example we
can introduce a predicate enabled-in, telling us when the first equation applies, by just giving
its lefthand side arguments as the predicate’s arguments:
op enabled-in : [Nat] [NatSet] -> [Bool] .
eq enabled-in(N, N ; NS) = true .
Note that we do not have to define when the enabled-in predicate is false. That is,
this predicate is really defined on the kind [Bool]. Our second owise equation is simply a
convenient shorthand for the conditional equation
ceq N in NS = false if enabled-in(N, NS) =/= true .
This is just a special case of a completely general theory transformation that translates
a specification containing equations with the owise attribute into a semantically equivalent
specification with no such attributes at all. A somewhat subtle aspect of this transformation13
is the interaction between owise equations and the operator evaluation strategies discussed
in Section 4.4.7. Suppose that an owise equation was used in defining the semantics of an
operator f. If f was (implicitly or explicitly) declared with a strategy, say,
f : s1 ...sn →s [strat (i1 ...ik0)].
then, the enabledf predicate should be defined with the same strategy,
enabledf : [s1]...[sn] →[Bool] [strat (i1 ...ik0)].
This will make sure that the reduction of f’s arguments prior to applying equations for f—
including the equations that will be introduced in our transformation to replace the owise
equations—takes place in exactly the same way for f and for enabledf, so that failure of
matching the normal equations is correctly captured by the failure of the enabledf predicate.
Furthermore, as we shall see, after the failure of matching the non-owise equations, the match-
ing substitution obtained when we apply the desugared version of an owise equation will then
properly take into account the evaluation of those arguments of f specified by f’s evaluation
strategy.
In general, if we are defining the equational semantics of an operation f : s1 ...sn →s and
we have given a partial definition of that operation by (possibly conditional) equations
f(u1
1,...,u1
n) = t1 if C1
...
f(um
1 ,...,um
n ) = tm if Cm
13We thank Joseph Hendrix for pointing out this subtlety.

<!-- manual page 80 -->
68 CHAPTER 4. FUNCTIONAL MODULES
then we can give one or more owise equations defining the function in the remaining cases by
means of equations of the form
f(v1
1,...,v1
n) = t′
1 if C′
1 [owise]
...
f(vk
1 ,...,vk
n) = t′
k if C′
k [owise]
Wecanviewsuchowiseequationsasshorthandnotationforcorrespondingordinaryconditional
equations of the form
f(y1,...,yn) = t′
1 if enabledf(y1,...,yn) ̸= true
∧ enabledf(v1
1,...,v1
n) := enabledf(y1,...,yn)
∧ C′
1
...
f(y1,...,yn) = t′
k if enabledf(y1,...,yn) ̸= true
∧ enabledf(vk
1 ,...,vk
n) := enabledf(y1,...,yn)
∧ C′
k
where the variables y1,...,yn are fresh new variables not appearing in any of the above owise
equations, and with yi of kind [si], 1 ≤ i ≤ n. All this assumes that in the transformed
specification we have declared the predicate enabledf : [s1]...[sn] →[Bool], with the same
evaluation strategy as f. Note the somewhat subtle use of the matching equations (see Sec-
tion 4.3) enabledf(vj
1,...,vj
n) := enabledf(y1,...,yn), 1 ≤j ≤k, in the conditions. Since
f and enabledf have the same strategy, after the arguments of the matching instance of the
expression enabledf(y1,...,yn) become evaluated according to the strategy, we are then able to
match enabledf(vj
1,...,vj
n) to that result, obtaining the desired substitution for the variables
of the lefthand side of the jth owise equation. That is, we obtain the same substitution as the
one we would have obtained matching f(vj
1,...,vj
n) to the same subject term after its subterms
under f had been evaluated according to f’s strategy.
Of course, the semantics of the enabledf predicate is defined in the expected way by the
equations
enabledf(u1
1,...,u1
n) = true if C1.
...
enabledf(um
1 ,...,um
n ) = true if Cm.
Thepossibilityofusingmultipleowiseequationsallowsustosimplifydefinitionsoffunctions
defined by cases on data with nested structure. Here is a simple, if silly, example in which the
sort Rhas elements a(n)and b(n), for natural numbers n, and the sort Shas elements g(r)and
h(r),withrofsortR.Theoperationftreatsconstructorsgandhdifferently,distinguishingonly
whether the subterm of sort R is constructed by a or not. Again, the predefined module NAT of
naturalnumbers(Section8.3)isimportedbymeansofaprotectingdeclaration(Section7.1.1).
fmod OWISE-TEST1 is
protecting NAT .
sorts R S .
op f : S Nat -> Nat .
ops g h : R -> S .

<!-- manual page 81 -->
4.5. STATEMENT ATTRIBUTES 69
ops a b : Nat -> R .
var r : R .
vars m n : Nat .
eq f(g(a(m)), n) = n .
eq f(h(a(m)), n) = n + 1 .
eq f(g(r), n) = 0 [owise] .
eq f(h(r), n) = 1 [owise] .
endfm
The four cases are illustrated by the following reductions.
Maude> red f(g(a(0)), 3) .
result NzNat: 3
Maude> red f(g(b(0)), 3) .
result Zero: 0
Maude> red f(h(b(0)), 3) .
result NzNat: 1
Maude> red f(h(a(0)), 3) .
result NzNat: 4
The subtle interaction between owise equations and operator evaluation strategies can be
illustrated by the following example:
fmod OWISE-TEST2 is
sort Foo .
ops a b c d : -> Foo .
op f : Foo -> Foo [strat (0 1 0)] .
op g : Foo -> Foo [strat (0)] .
var X : Foo .
eq b = c .
eq f(a) = d .
eq f(X) = g(X) [owise] .
endfm
Now consider the term f(b). Intuitively, one could expect that, given that the first equation
for f cannot be applied to this term, the owise equation is applied obtaining the term g(b),
and this is then expected to be the final result of the reduction, because the strategy (0) for g
forbids evaluating its argument. However, as we can see in the following reduction, this is not
the case.
Maude> red f(b) .
result Foo: g(c)
The result is g(c), because the owise equation is not considered until after evaluating the
final 0 in the strategy for f, and by then f(b) is simplified to f(c) as instructed by the 1 in
such strategy; then, the owise equation applied to f(c) produces g(c).
It can be interesting to consider the semantically equivalent transformed specification:
fmod OWISE-TEST2-TRANSFORMED is
sort Foo .
ops a b c d : -> Foo .
op f : Foo -> Foo [strat (0 1 0)] .

<!-- manual page 82 -->
70 CHAPTER 4. FUNCTIONAL MODULES
op enabled-f : Foo -> Bool [strat (0 1 0)] .
op g : Foo -> Foo [strat (0)] .
vars X Y : Foo .
eq b = c .
eq f(a) = d .
eq enabled-f(a) = true .
ceq f(Y) = g(X)
if enabled-f(Y) =/= true /\ enabled-f(X) := enabled-f(Y) .
endfm
Maude> red f(b) .
result Foo: g(c)
where, as pointed out in our comments on the general transformation, the fact that enabled-f
has the same strategy as f and the use of the matching equation
enabled-f(X) := enabled-f(Y)
are crucial for obtaining a semantically equivalent specification.
4.5.5 Print
The print attribute allows the user to specify information to be printed when a statement
(equation, membership axiom, or rule) is executed. A print attribute declaration looks like
eq f(X) = b [print "X = " X] .
The keyword print is followed by a possibly empty list of items where each item is either
a string constant or a variable. Mentioned variables must actually occur in the statement. If
a non-occurring variable appears as a print item, it will be pruned and Maude will issue a
warning.
Here is an example that uses the print attribute to track calls to a recursive function that
reverses a list.
fmod PRINT-ATTRIBUTE-EX is
sorts Foo FooList .
ops a b : -> Foo [ctor] .
subsort Foo < FooList .
op nil : -> FooList [ctor] .
op __ : FooList FooList -> FooList [ctor assoc id: nil] .
op reverse : FooList -> FooList .
eq reverse(nil) = nil .
eq reverse(foo:Foo fl:FooList) = reverse(fl:FooList) foo:Foo
[print "first = " foo:Foo ", rest = " fl:FooList] .
endfm
Maude will only use the print attribute in print attribute mode, which is off by default.
Thus to run the above example (after loading it into Maude) it is necessary to execute the
following command (see Appendix A.9).
Maude> set print attribute on .
Then reducing the expression reverse(a b a b) results in the following output:

<!-- manual page 83 -->
4.6. ADMISSIBLE FUNCTIONAL MODULES 71
Maude> red in PRINT-ATTRIBUTE-EX : reverse(a b a b) .
reduce in PRINT-ATTRIBUTE-EX : reverse(a b a b) .
first = a, rest = b a b
first = b, rest = a b
first = a, rest = b
first = b, rest = nil
rewrites: 5 in 0ms cpu (0ms real) (18587 rewrites/second)
result FooList: b a b a
The print attribute is an alternative to tracing (see Section 21.1.1) to find out which
statements Maude is executing. It allows the user control of what information is printed. It is
also a nice way to show what is going on in demos.
4.6 Admissible functional modules
The nonexec attribute allows us to include arbitrary equations or memberships, conditional or
not, in a functional module and likewise in a functional theory (see Section 7.4.1). Any such
statement is then disregarded for purposes of execution by the Maude engine: it can only be
used in a controlled way at the metalevel. But what about all the other statements? That is,
what requirements should be imposed on executable equations and memberships so that they
can be given an operational interpretation and can be executed by the Maude engine?
The intuitive idea is that we want to use such equations as simplification rules from left
to right to reach a single final result or canonical form. For this purpose, the executable
equations and memberships (that is, all statements not having the nonexec attribute) should
be Church-Rosser and terminating (modulo the equational attributes declared in the module)
in the sense explained in Section 4.7 below. This guarantees that, given a term t, all chains of
equationalsimplificationusingthoseequationsandmembershipsendinauniquecanonicalform
(again, modulo the equational attributes). Furthermore, under the preregularity assumption
(see Section 3.8), such a canonical form has the smallest sort possible in the subsort ordering.
The traditional requirement in this context is that, given a conditional equation14 t=
t′ if C1 ∧...∧Cn, the set of variables appearing in tcontains those appearing in both t′and
in the conditions Ci. In Maude, this requirement is relaxed to support matching equations in
conditions (see Section 4.3) which can introduce new variables not present in t. Specifically, all
executable conditional equations in a Maude functional module M have to satisfy the following
admissibility requirements, ensuring that all the extra variables will become instantiated by
matching:
1.
vars(t′) ⊆vars(t) ∪
n
vars(Cj).
j=1
2. If Ci is an equation ui = u′
i or a membership ui : s, then
vars(Ci) ⊆vars(t) ∪
i−1
vars(Cj).
j=1
3. If Ci is a matching equation ui := u′
i, then ui is an M-pattern and
vars(u′
i) ⊆vars(t) ∪
i−1
vars(Cj).
j=1
14For the purposes of this discussion we can regard unconditional equations as the special case of conditional
equations with empty condition, or with the condition true= true.

<!-- manual page 84 -->
72 CHAPTER 4. FUNCTIONAL MODULES
In a similar way, all executable conditional memberships t: s if C1 ∧...∧Cn must satisfy
conditions 2–3 above.
In summary, therefore, we expect all executable equations and memberships in a functional
module (and also in a system module) to be Church-Rosser and terminating (see Section 4.7
below, and [14, Section 10.1]) and to satisfy the above admissibility requirements.
4.7 Matching and equational simplification
Although this section and the next are quite technical, and it may be possible to skip them in
a first reading, they introduce the concepts of matching and equational simplification that are
essential to understand how Maude works. Therefore, we advise the reader to come back to
them as needed to gain a better understanding of those concepts.
Recall from Section 4.3 that a functional module defines an equational theory (Σ,E∪A)
in membership equational logic, with A the equations specified as equational attributes in
operators (see Section 4.4.1), and E the (possibly conditional) equations and memberships
specified as statements.
GroundtermsinthesignatureΣ formaΣ-algebradenotedTΣ. Similarly, equivalenceclasses
of terms modulo E∪Adefine the Σ-algebra denoted TΣ/E∪A, which is the initial model for the
theory (Σ,E∪A) specified by the module [95].
Given a set X of variables, we can add them to the signature Σ as new constants, and get
in this way a term algebra TΣ(X) where now the terms may have variables in X.
Given a set X of variables, each having a given kind, a (ground) substitution is a kind-
preserving function σ: X −→TΣ. Such substitutions may be used to represent assignments of
terms in TΣ to the variables in X, and also assignments of elements in TΣ/E∪A to such variables
by σ picking up a representative of the corresponding E∪A-equivalence class. For example, a
very natural choice is to assign to each xin X a term σ(x) which is in canonical form according
to E∪A. Furthermore, under the preregularity, Church-Rosser, and termination assumptions
(more on this below) this canonical form will have a least sort. Therefore, we may allow each
variable xin X to have either a kind or a sort assigned to it, and can call the substitution σwell-
sorted relative to E∪Aif the least sort of σ(x) is smaller or equal to that of x. By substituting
terms for variables (as indicated by σ) in the usual way, a substitution σ: X −→TΣ is extended
to a homomorphic function on terms σ: TΣ(X) −→TΣ that we denote with the same name.
Given a term t∈TΣ(X), corresponding to the lefthand side of an oriented equation, and a
subject ground term u ∈TΣ, we say that t matches15 u if there is a substitution σ such that
σ(t) ≡u, that is, σ(t) and u are syntactically equal terms.
For an oriented Σ-equation l= r to be used in equational simplification, it is required that
all variables in the righthand side r also appear among the variables of the lefthand side l.
In the case of a conditional equation l= r if cond, this requirement is relaxed, so that more
variables can appear in the condition cond, provided that they are introduced by matching
equations according to the admissibility requirements in Section 4.6; then the variables in the
righthand side r must be among those in the lefthand side l or in the condition cond. Under
this assumption, given a theory (Σ,E) a term t rewrites to a term t′using such an equation if
there is a subterm t|p of t at a given position16 p of t such that l matches t|p via a well-sorted
15Some authors would instead say that u matches t.
16We can represent a term t as a tree, and use strings of numbers to identify positions p in the tree, thus
identifying subterms t|p. For example, for t= f(g(a),h(b)), we have t|nil = t, t|1 = g(a), t|11 = a, t|2 = h(b),
and t|21 = b.

<!-- manual page 85 -->
4.7. MATCHING AND EQUATIONAL SIMPLIFICATION 73
t
E ∗
????????
∗
E
t1
t2
∗
E
E
t′
∗
Figure 4.1: Confluence diagram
substitution17 σ and t′ is obtained from t by replacing the subterm t|p ≡σ(l) with the term
σ(r). In addition, if the equation has a condition cond, the substitution σ must make the
condition provably true according to the equations and memberships in E, which are assumed
to be Church-Rosser and terminating and are used also from left to right to try to simplify the
condition. Note that, in general, the variables instantiated by σ must contain both those in the
lefthand side, and those in the condition (which are incrementally matched using the matching
equations).
We denote this step of equational simplification by t →E t′, where the possible equations
for rewriting are chosen in the set E. The reflexive and transitive closure of the relation →E is
denoted →∗
E.
In many texts, equational simplification is also called (equational) rewriting but, since in
Maude we have two very different types of rewriting, rewriting with equations in functional
modules, and rewriting with rules in system modules, each with a completely different seman-
tics, to avoid confusion we favor the terminology of equational simplification for the process of
rewriting with equations.
A set of equations E is confluent when any two rewritings of a term can always be unified
by further rewriting: if t →∗
E t1 and t →∗
E t2, then there exists a term t′ such that t1 →∗
E t′
and t2 →∗
E t′. This is summarized in Figure 4.1.
A set of equations E is terminating when there is no infinite sequence of rewriting steps
t0 →E t1 →E t2 →E...
If E is both confluent and terminating, a term tcan be reduced to a unique canonical form
t↓E, that is, to a unique term that can no longer be rewritten. Therefore, in order to check
semantic equality of two terms t= t′(that is, that they belong to the same equivalence class),
it is enough to check that their respective canonical forms are equal, t↓E = t′↓E, but, since
canonical forms cannot be rewritten anymore, the last equality is just syntactic coincidence:
t↓E ≡t′↓E.
In membership equational theories a third important property is sort decreasingness. In-
tuitively, this means that, assuming E is confluent and terminating, the canonical form t↓E
of a term t by the equations E should have the least sort possible among the sorts of all the
terms equivalent to it by the equations E; and it should be possible to compute this least
sort from the canonical form itself, using only the operator declarations and the memberships.
By a Church-Rosser and terminating theory (Σ,E) we precisely mean one that is confluent,
terminating, and sort-decreasing. For a more detailed treatment of these properties, we refer
the reader to the paper [14].
17Note that if a variable xhas a sort sinstead of a kind, well sortedness of σ means that σ(x) must provably
have sort s (or lower) according to the equations E.

<!-- manual page 86 -->
74 CHAPTER 4. FUNCTIONAL MODULES
Since Maude functional modules have an initial algebra semantics, we are primarily in-
terested in ground terms. Therefore, we can relax the above Church-Rosser and termination
requirements by requiring that they just hold for ground terms, without losing the desired coin-
cidence between the mathematical and operational semantics. In this way, we obtain notions of
ground Church-Rosser, terminating, confluent, etc. specifications. In practice, some perfectly
reasonable Maude functional modules are ground confluent, but fail to be confluent. This how-
ever is not a problem, since ground confluence (together with ground termination) is just what
is needed to ensure uniqueness of canonical forms. Indeed, under the ground Church-Rosser
and termination assumptions, it is easy to prove that we have the desired isomorphism
TΣ/E
∼
= CanΣ/E
ensuring the coincidence between the mathematical semantics of (Σ,E) provided by the initial
algebraTΣ/E, anditsoperational semantics byequationalsimplificationprovidedbythealgebra
CanΣ/E of canonical forms.
Equational specifications (Σ,E) in Maude functional modules (and in the equational part of
system modules), are assumed to be ground Church-Rosser and terminating up to the context-
sensitive strategy specified by the evaluation strategies declared for the operators in Σ (see
Section 4.4.7). More precisely, we can view the information about operator evaluation strate-
gies as a function µ that assigns to each operator f ∈Σ with n argument sorts a string of
numbers indicating the argument positions to be evaluated and ended with a 0 (that is, the in-
formation given in the operator’s strat attribute, or, if no such information is given, the string
1 ...n0). This then defines a more restricted rewrite relation →µ
E where we can only rewrite
in subterms in positions that can be evaluated according to µ. If the relation →µ
E is (ground)
confluent, we call the specification (ground) µ-confluent; similarly, if →µ
E is (ground) terminat-
ing, we call it (ground) µ-terminating. We define the concepts of (ground) µ-sort-decreasing
and (ground) µ-Church-Rosser in the same way. When we talk about the specification be-
ing “ground Church-Rosser and terminating up to the context-sensitive strategy specified by
the evaluation strategies,” we exactly mean that it is ground µ-Church-Rosser and ground
µ-terminating. Of course, when no such strategies are declared, this specializes to the usual
notions of ground Church-Rosser and ground terminating. Under the ground µ-Church-Rosser
and ground µ-terminating assumptions, the µ-canonical forms define a canonical term algebra
Canµ
Σ/E (see [71]), which provides a perfect mathematical model for the module’s operational
semantics, since its elements are the values that the user gets when evaluating expressions in
such a module. The question then arises: how is this model related to the module’s mathe-
matical semantics? In general, the quotient map t→[t]E sending each µ-canonical form to its
E-equivalence class is a surjective homomorphism
q: Canµ
Σ/E −→TΣ/E,
but not necessarily an isomorphism. If qfails to be an isomorphism, this means that µ-rewriting
is a sound deductive method for proving E-equalities, but it is incomplete. Therefore we call the
specification µ-semantically complete iff q is an isomorphism. µ-semantic completeness there-
fore expresses the complete agreement between the mathematical and operational semantics of
the module. Specifications where evaluation strategies are used mainly for efficiency and/or
termination purposes, that is, those where the execution becomes more efficient by avoiding
wasteful computation in unnecessary parts of the term and/or that would not terminate with-
out the strategy restrictions are typically µ-semantically complete. Instead, specifications such
as the sieve of Eratosthenes in Section 4.4.7, where the main intent is to compute with infinite
data structures in a lazy way, are typically µ-semantically incomplete. Not all is lost in this
second case: we still have a good mathematical model associated to our specification, namely,

<!-- manual page 87 -->
4.8. MORE ON MATCHING AND SIMPLIFICATION MODULO 75
Canµ
Σ/E, but this is a more concrete model than TΣ/E, that is, one in which fewer elements are
identified.
What are the appropriate notions when we have a theory of the form (Σ,E ∪A)? Then
matching must be defined modulo the equational axioms A, and all the above concepts, in-
cluding those for µ-rewriting, must be generalized to equational simplification, confluence, and
termination modulo A. We discuss this in more detail in Section 4.8 below. See also [71] for a
detailed treatment of µ-rewriting and µ-semantic completeness modulo axioms A.
As already mentioned, the operational semantics of functional modules is equational sim-
plification, that is, equational rewriting of terms until a canonical form is obtained in the sense
explained above. Notice that the system does not check the ground confluence and termination
properties: theyarelefttotheuser’sresponsibility. However,insomecasesitispossibletocheck
these properties with Maude’s Church-Rosser checker and termination tools [25, 43, 39, 38].
Similar checkings are also possible for functional modules with evaluation strategies; for exam-
ple, the Maude’s MTT termination tool can check µ-termination (also called context-sensitive
termination [83]). Moreover, although the relations between the standard Church-Rosser prop-
erty and the µ-Church-Rosser property are somewhat subtle [82, 84], the work in [71] shows how
one can use standard tools in conjunction with Maude’s Sufficient Completeness Checker [73]
to check both the µ-Church-Rosser property and µ-semantic completeness. See Section 11.4
for some examples of the use of these formal tools.
4.8 More on matching and simplification modulo
In the Maude implementation, rewriting modulo Ais accomplished by using a matching modulo
A algorithm. More precisely, given an equational theory A, a term t (corresponding to the
lefthand side of an equation) and a subject term u, we say that t matches u modulo A (or
that t A-matches u) if there is a substitution σ such that σ(t) =A u, that is, σ(t) and u are
equal modulo the equational theory A (compare with the syntactic definition of matching in
Section 4.7 above).
Given an equational theory A= ∪iAfi corresponding to all the attributes declared in
different binary operators, Maude synthesizes a combined matching algorithm for the theory
A, and does both equational simplification (with equations) and rewriting (with rules in system
modules, see Chapter 5) modulo the axioms A.
Note, however, that for operators f whose equational axioms A include the associativity
axiom, to achieve the effect of simplification modulo A using an A-matching algorithm, we
have to attempt matching a lefthand side of the form f(t1, t2) not only on a subject term u,
but also on all its f-subterms, that is, on those “fragments” of the top structure of the term
that could be matched by f(t1, t2). For example, assuming a binary associative operator f
and constants a, b, c, and d of the appropriate sort, the term t= f(a,b) does not match the
term u = f(a, f(b, f(c, d))), that is, there is no substitution making both terms equal
modulo associativity; however, because of associativity of f, u is equivalent to f(f(a, b),
f(c, d)) and then t trivially matches the first subterm. This becomes easier to see using
mixfix notation; if f=
_._, then t= a . b and u= a . b . c . d, and we clearly see that t
matches a fragment of u. For the case where the only axiom is associativity, the_._-subterms
of a . b . c . d are
a . b
a . b . c
b . c
b . c . d
c . d

<!-- manual page 88 -->
76 CHAPTER 4. FUNCTIONAL MODULES
If the operation_._ had been declared both associative and commutative, then we should add
to those the additional subterms
a . c
a . d
b . d
a . b . d
a . c . d
If the term f(t1, t2) matches either u or an f-subterm of u modulo A, then we say that
f(t1, t2) matches u with extension modulo A (or that f(t1, t2) A-matches u with extension).
For example, the lefthand side of the equation a . b = e matches a . b . c . d with exten-
sion modulo associativity, and the lefthand side of a . d = g matches a . b . c . d with
extension modulo associativity and commutativity.
For f a binary operator with equational attributes Af including the associativity axiom, we
now define how a subject term uis Af-rewritten with extension using an equation f(t1, t2) = v.
First of all, f(t1, t2) must Af-match with extension a maximal f-subterm w of u (that is, an
f-subterm of u that is not itself an f-subterm of a bigger f-subterm). This means that there
is an f-subterm w0 of w and a substitution σ such that σ(f(t1, t2)) =Af w0. Then, the
corresponding Af-rewriting with extension step rewrites u to the term obtained by replacing
the subterm w0 by σ(v).
Note that a term f(t1, t2) Af-matches with extension a maximal f-subterm if and only if it
Af-matches without extension some f-subterm. This is of course the important practical ad-
vantage of A-matching and A-rewriting with extension, namely, that only maximal f-subterms
of a term have to be inspected to get the effect of rewriting equivalence classes modulo A. For
more technical details on rewriting modulo a set of axioms, see, e.g., [33].
Matching with extension for an associative operator essentially corresponds to matching
without extension for a collection of associated equations. For example, we could have “gener-
alized” the equation a . b = e with _._ associative to the equations
eq a . b = e .
eq X . a . b = X . e .
eq a . b . Y = e . Y .
eq X . a . b . Y = X . e . Y .
so that we could have achieved the same effect by rewriting only at the top of maximal f-
subterms (without extension). Similarly, for_._ associative and commutative, we could have
generalized the same equation a . b = e to the equations
eq a . b = e .
eq a . b . Y = e . Y .
In Maude this generalization does not have to be performed explicitly as a transformation of
the specification. It is instead achieved implicitly in a built-in way by performing A-matching
with extension. If the equational axioms declared for a binary operator f include the asso-
ciativity axiom, then a subject term u with f as its top operator is internally represented
(but this representation can also be externally used, see Section 3.9.3) as the flattened term
f(u1, ..., un), with the u1, ..., un having top operators different from f. Furthermore, if a
(two-sided) identity element e has been declared for f, then ui ̸= e, 1 ≤i ≤n. That is, we
assume in this case that all identities have been simplified away.
Relative to this internal representation, it is then easy to define the notion of an f-subterm.
If the axioms of f include associativity but not commutativity, then the f-subterms of the term
f(u1, ..., un) are all terms of the form f(uk, ..., uk+h) with 1 ≤k≤n−1 and 1 ≤h≤n−k.

<!-- manual page 89 -->
4.8. MORE ON MATCHING AND SIMPLIFICATION MODULO 77
Similarly, if the axioms of f include associativity and commutativity, then the f-subterms
of f(u1, ..., un) are all terms of the form f(uk1 , ..., ukh ) with 1 ≤ki1 <···<kih ≤n, and
2 ≤h≤n.
The concepts of positions in a term and depth of a term, that are important in many situa-
tions, refer to this flattened form. The compact notation for terms constructed with operators
having the iter attribute (Section 4.4.2) is also considered a form of flattened notation, so
that, for the purpose of calculating term depth, if the top level is at level 0, then the occurrence
of X:Foo in f^3(X:Foo) is at level 1, not level 3.
Adding axioms for an identity element e to a possibly associative and/or commutative
operation f leads to some subtle cases, where the proper application of the general notions
may not always coincide with the user’s expectations. To begin with, unexpected cases of
nontermination may be introduced by an unwary user. For example, the equation
eq a . X = b . a .
will cause nontermination when _._ is declared associative with identity 1, since we have, for
example,
d . a → d . b . a
→ d . b . b . a
.
.
.
→ d . b . b .··· . b . a
.
.
.
by instantiating each time the variable X to the identity element 1.
A second source of unexpected behavior is the fact that a lefthand side involving an as-
sociative operator may, in the presence of an additional identity attribute, match a term not
involving at all that operator. Thus, for the above equation, we have also the nonterminating
chain of rewriting steps
a → b . a
→ b . b . a
→ b . b .··· . b . a
.
.
.
.
.
.
In a similar way, in the presence of an identity element, the user’s expectations about when
a lefthand side will match with extension a subject term may not fully agree with the proper
technical definition. Consider, for example, a binary operation_._ that is associative and
commutative, and that has an identity element 1, and let
eq a . X = c .
be an equation. Then,
1. The lefthand side a . X matches the subject term a modulo the axioms by instantiating
X to 1, giving rise to the simplification
a → c.

<!-- manual page 90 -->
78 CHAPTER 4. FUNCTIONAL MODULES
2. The same lefthand side matches the subject term a . b . c with extension in three
different ways, namely, with substitutions X →b . c, X →b, and X →c, giving rise to
the three simplifications
a . b . c → c
a . b . c → c . c
a . b . c → b . c
3. For the same subject term a . b . c, the substitution X → 1 is not a match with
extension of the above lefthand side, because the term a . 1 is not a _._-subterm of
the term a . b . c. However, because of item 1 above, we know that the equation will
match that way not at the top, but “one level down,” leading to the simplification
a . b . c → c . b . c
It is also important to realize that there is no match with extension between the lefthand
side of the equation a = b and the subject term a . b . c (because the lefthand side a is not
a _._-term), although again the equation will match that way not at the top, but “one level
down,” leading to the simplification
a . b . c → b . b . c
Of course, lefthand sides may contain several operators, each matched modulo a different
theory. Maude will then match each fragment of a lefthand side according to its given theory.
Consider, for example, the following specification where_._ is associative and _+_ is asso-
ciative and commutative:
fmod XMATCH-TEST is
sort Elt .
ops a b c d e : -> Elt .
op _._ : Elt Elt -> Elt [assoc] .
op _+_ : Elt Elt -> Elt [assoc comm] .
vars X Y Z : Elt .
eq X . (Y + Z) = (X . Y) + (X . Z) [metadata "distributivity"] .
endfm
The lefthand side of the distributivity equation will produce 12 matches with extension for
the subject term
a . b . (c + d + e)
Enumerating these by hand would be tedious and error prone, however Maude provides the
xmatch command (see also Appendix A.3) for just this purpose:
xmatch X . (Y + Z) <=? a . b . (c + d + e) .
The output given by Maude consists of the substitution for each match with extension
together with the portion of the subject actually matched:
Maude> xmatch X . (Y + Z) <=? a . b . (c + d + e) .
xmatch in XMATCH-TEST : X . Z + Y <=? a . b . c + d + e .
Decision time: 0ms cpu (0ms real)
Solution 1
Matched portion = (whole)
X:Elt --> a . b

<!-- manual page 91 -->
4.8. MORE ON MATCHING AND SIMPLIFICATION MODULO 79
Y:Elt --> c
Z:Elt --> d + e
Solution 2
Matched portion = b . (c + d + e)
X:Elt --> b
Y:Elt --> c
Z:Elt --> d + e
Solution 3
Matched portion = (whole)
X:Elt --> a . b
Y:Elt --> d
Z:Elt --> c + e
Solution 4
Matched portion = b . (c + d + e)
X:Elt --> b
Y:Elt --> d
Z:Elt --> c + e
Solution 5
Matched portion = (whole)
X:Elt --> a . b
Y:Elt --> e
Z:Elt --> c + d
Solution 6
Matched portion = b . (c + d + e)
X:Elt --> b
Y:Elt --> e
Z:Elt --> c + d
Solution 7
Matched portion = (whole)
X:Elt --> a . b
Y:Elt --> c + d
Z:Elt --> e
Solution 8
Matched portion = b . (c + d + e)
X:Elt --> b
Y:Elt --> c + d
Z:Elt --> e
Solution 9
Matched portion = (whole)
X:Elt --> a . b
Y:Elt --> c + e
Z:Elt --> d
Solution 10
Matched portion = b . (c + d + e)
X:Elt --> b

<!-- manual page 92 -->
80 CHAPTER 4. FUNCTIONAL MODULES
Y:Elt --> c + e
Z:Elt --> d
Solution 11
Matched portion = (whole)
X:Elt --> a . b
Y:Elt --> d + e
Z:Elt --> c
Solution 12
Matched portion = b . (c + d + e)
X:Elt --> b
Y:Elt --> d + e
Z:Elt --> c
Note that extension is only used for matching the top operation,_._ in this example, but
not _+_. This is the reason why the subterm Y + Z of the lefthand side should match the entire
maximal _+_-subterm of the subject term, and not just some _+_-subterm.
Foroperatorswiththeiterattribute, thesituationwithmatchingisanalogoustotheassoc
theory, so that proper subterms of say f^3(X:Foo), such as f^2(X:Foo) and f(X:Foo), can
also be matched by means of extension.
4.9 The reduce, match, trace, and show commands
Here we assemble the whole module for the NUMBERS running example to illustrate some of the
basic commands for interacting with Maude. See Appendix A for full details about these and
other Maude commands.
Notice that, since the result of the _in_ predicate is a Boolean value, we import the pre-
defined module BOOL (see Section 8.1) by means of a protecting declaration (described in
Section 7.1.1).
fmod NUMBERS is
protecting BOOL .
sort Zero .
sorts Nat NzNat .
subsort Zero NzNat < Nat .
op zero : -> Zero [ctor] .
op s_ : Nat -> NzNat [ctor] .
op sd : Nat Nat -> Nat .
ops _+_ _*_ : Nat Nat -> Nat [assoc comm] .
op _+_ : NzNat Nat -> NzNat [ditto] .
op _*_ : NzNat NzNat -> NzNat [ditto] .
op p : NzNat -> Nat .
vars I N M : Nat .
eq N + zero = N .
eq N + s M = s (N + M) .
eq sd(N, N) = zero .
eq sd(N, zero) = N .
eq sd(zero, N) = N .
eq sd(s N, s M) = sd(N, M) .
eq N * zero = zero .

<!-- manual page 93 -->
4.9. THE REDUCE, MATCH, TRACE, AND SHOW COMMANDS 81
eq N * s M = (N * M) + N .
eq p(s N) = N [label partial-predecessor] .
eq (N + M) * I = (N * I) + (M * I)
[nonexec metadata "distributive law"] .
sort Nat3 .
ops 0 1 2 : -> Nat3 [ctor] .
op _+_ : Nat3 Nat3 -> Nat3 [comm] .
vars N3 : Nat3 .
eq N3 + 0 = N3 .
eq 1 + 1 = 2 .
eq 1 + 2 = 0 .
eq 2 + 2 = 1 .
sort NatSeq .
subsort Nat < NatSeq .
op nil : -> NatSeq .
op __ : NatSeq NatSeq -> NatSeq [assoc id: nil] .
sort NatSet .
subsort Nat < NatSet .
op empty : -> NatSet .
op _;_ : NatSet NatSet -> NatSet [assoc comm id: empty] .
eq N ; N = N [label natset-idem] .
op _in_ : Nat NatSet -> Bool .
var NS : NatSet .
eq N in N ; NS = true .
eq N in NS = false [owise] .
endfm
First, we evaluate some expressions using the reduce command. Maude repeats the com-
mand filling in any omitted optional information. Then statistics about the execution are
printed.18 Finally, the result is printed, prefaced by its least sort.
The first two examples evaluate the sum of three ones in Nat and in Nat3.
Maude> red s zero + s zero + s zero .
reduce in NUMBERS : s zero + s zero + s zero .
rewrites: 4 in 0ms cpu (0ms real) (~ rews/sec)
result NzNat: s s s zero
Maude> red 1 + (1 + 1) .
reduce in NUMBERS : 1 + (1 + 1) .
rewrites: 2 in 0ms cpu (0ms real) (~ rews/sec)
result Nat3: 0
The next example illustrates the effect of the idempotency equation for sets of natural
numbers.
Maude> red zero ; s zero ; zero ; s zero .
reduce in NUMBERS : zero ; s zero ; zero ; s zero .
rewrites: 2 in 0ms cpu (0ms real) (~ rews/sec)
18The cpu and real time information is not printed if the user has made use of the set show timing off
command (see Appendix A.11).

<!-- manual page 94 -->
82 CHAPTER 4. FUNCTIONAL MODULES
result NatSet: zero ; s zero
Finally we convince ourselves that the owise attribute works.
Maude> red zero in s zero ; zero ; s s zero .
reduce in NUMBERS : zero in s zero ; zero ; s s zero .
rewrites: 1 in 0ms cpu (0ms real) (~ rews/sec)
result Bool: true
Maude> red zero in s zero ; s s zero .
reduce in NUMBERS : zero in s zero ; s s zero .
rewrites: 1 in 0ms cpu (0ms real) (~ rews/sec)
result Bool: false
The commands xmatch and match operate in the same way, unless the subject term has
an operator that needs extension on top, in which case it can match proper subterms in the
same theory layer, as required for rewriting modulo that theory. The xmatch command was
illustrated in Section 4.8. Here we compare match and xmatch on a pattern that splits a
sequence of natural numbers into two parts. To be safe, we ask for at most five matches, but
in fact there are only four.
Maude> match [5] NS0:NatSeq NS1:NatSeq <=? zero zero zero .
match [5] in NUMBERS : NS0:NatSeq NS1:NatSeq <=? zero zero zero .
Decision time: 0ms cpu (0ms real)
Solution 1
NS0:NatSeq --> nil
NS1:NatSeq --> zero zero zero
Solution 2
NS0:NatSeq --> zero
NS1:NatSeq --> zero zero
Solution 3
NS0:NatSeq --> zero zero
NS1:NatSeq --> zero
Solution 4
NS0:NatSeq --> zero zero zero
NS1:NatSeq --> nil
Using the xmatch command for the same pattern and term, we see that in addition to the
whole term matches, Maude also reports matches within the subterm zero zero. In fact, there
are two occurrences of this subterm. We only show five of the matches.
Maude> xmatch [5] NS0:NatSeq NS1:NatSeq <=? zero zero zero .
xmatch [5] in NUMBERS : NS0:NatSeq NS1:NatSeq <=? zero zero zero .
Decision time: 0ms cpu (7ms real)
Solution 1
Matched portion = zero zero
NS0:NatSeq --> nil
NS1:NatSeq --> zero zero
Solution 2
Matched portion = zero zero

<!-- manual page 95 -->
4.9. THE REDUCE, MATCH, TRACE, AND SHOW COMMANDS 83
NS0:NatSeq --> zero
NS1:NatSeq --> zero
Solution 3
Matched portion = zero zero
NS0:NatSeq --> zero zero
NS1:NatSeq --> nil
Solution 4
Matched portion = (whole)
NS0:NatSeq --> nil
NS1:NatSeq --> zero zero zero
Solution 5
Matched portion = (whole)
NS0:NatSeq --> zero
NS1:NatSeq --> zero zero
Let us consider now a small example using the trace command. We turn on selective
tracing and choose to trace only uses of the equation labeled partial-predecessor.
Maude> set trace on .
Maude> set trace select on .
Maude> trace select partial-predecessor .
Maude> red s s p(s zero) + s p(s zero) .
reduce in NUMBERS : s s p(s zero) + s p(s zero) .
*********** equation
eq [partial-predecessor] : p(s N) = N .
N --> zero
p(s zero)
--->
zero
rewrites: 3 in 0ms cpu (0ms real) (~ rews/sec)
result NzNat: s s s zero
Note that Maude only reports one use of this equation, despite the fact that there are two
occurrences in the term. This is because, when performing equational simplification, occur-
rences of the same subterm are internally shared19 and hence there is only one occurrence of
the subterm p(s zero) in the internal representation.
We can ask Maude to show the module FIBONACCI (assuming it has been loaded).
Maude> show module FIBONACCI .
fmod FIBONACCI is
protecting NAT .
op fibo : Nat -> Nat [memo] .
var N : Nat .
19However, this sharing—i.e., treating the term as a dag instead of as a tree—is not done in a maximal way, so
that all subterms that can be shared are; instead, term sharing is itself introduced incrementally by equational
simplification, since Maude analyzes righthand sides of equations to identify its shared subterms. As explained
by Eker in [50], in the presence of operator evaluation strategies (Section 4.4.7) term sharing has to be done
carefully. Furthermore, when rewriting is performed with a rule in a system module (see Chapter 5), rather
than with an equation, Maude will incrementally unshare those parts of the subject term needed to ensure that
all possible rewrite with rules are considered. This is because rules in system modules need not be confluent.
As a consequence, two identical subterms could be rewritten in totally different ways; but this of course would
be prevented if they were to be shared.

<!-- manual page 96 -->
84 CHAPTER 4. FUNCTIONAL MODULES
eq fibo(0) = 0 .
eq fibo(1) = 1 .
eq fibo(s s N) = fibo(N) + fibo(s N) .
endfm
The show sorts command shows all the sorts declared and for each sort its sub- and super-
sorts.
Maude> show sorts NUMBERS .
sort Bool .
sort Zero . subsorts Zero < Nat NatSet NatSeq .
sort Nat . subsorts NzNat Zero < Nat < NatSet NatSeq .
sort NzNat . subsorts NzNat < Nat NatSet NatSeq .
sort Nat3 .
sort NatSeq . subsorts NzNat Zero Nat < NatSeq .
sort NatSet . subsorts NzNat Zero Nat < NatSet .
Theshow componentscommandshowstheconnectedcomponents(kinds)inthesortpartial
order.
Maude> show components NUMBERS .
[Bool]:
1 Bool
[NatSeq, NatSet]:
1 NatSeq
2 NatSet
3 Nat
4 Zero
5 NzNat
[Nat3] (error free):
1 Nat3
Note that the name of the kind corresponding to the connected component containing the
naturalnumberscontainsthenamesoftwosorts. Thesearethemaximalsortsinthecomponent.
The (error free) comment about the sort Nat3 means that all terms of kind [Nat3] are in
fact of sort Nat3.

