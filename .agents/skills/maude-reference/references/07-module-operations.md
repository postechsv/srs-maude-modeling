# Chapter 7: Module Operations (manual pages 143-196)

<!-- manual page 143 -->
Chapter 7
Module Operations
Specifications and code should be structured in modules of relatively small size to facilitate
understandability of large systems, increase reusability of components, and localize the effects
of system changes. In Maude, these goals are achieved by means of a module algebra that
supports parameterized programming techniques in the OBJ3 style [69] as well as the definition
of module hierarchies, i.e., acyclic graphs of module importations; that is, each functional or
system module can import other Maude modules as submodules. Since the submodule relation
is transitive, we can in this way develop module hierarchies. Mathematically, we can think
of such hierarchies as partial orders of theory inclusions, that is, the theory of the importing
module contains the theories of its submodules as subtheories.
As in Clear [17], OBJ [69], and other specification languages in that tradition, the abstract
syntax for writing specifications in Maude can be seen as given by module expressions, where
the notion of module expression is understood as an expression that defines a new module
out of previously defined modules by combining and/or modifying them according to a spe-
cific set of operations. In fact, structuring is essential in all specification languages, not only
to facilitate the construction of specifications from already existing ones—with more or less
flexible reusability mechanisms—but also for managing the complexity of understanding and
analyzing large specifications. Maude supports module operations for summation, renaming,
and instantiation of parameterized modules.
Section 7.1 introduces module importations and the different modes in which such impor-
tations can take place. Sections 7.2 and 7.3 discuss, respectively, the summation and renaming
module expressions. Section 7.4 introduces parameterized programming, including the use of
theories and views, the parameterization of modules, and the instantiation of parameterized
modules. We refer to [37, 41, 42] for a deeper discussion on the semantics of the Maude module
operations.
7.1 Module importation
Recall that a functional module M specifies a membership equational theory of the form
(Σ,E∪A), with Σ its signature, A the equational attributes specified for its operators, and
E its set of equations and memberships. A submodule M′ of M is either a module directly
imported by M, or a submodule of one of the modules directly imported by M. Then M′spec-
ifies a membership equational subtheory (Σ′,E′∪A′) ⊆(Σ,E∪A). Specifically, we have three
inclusions: Σ′⊆Σ, E′⊆E, and A′⊆A. Furthermore, since in Maude subsort-overloaded op-
erators must have the same equational attributes, Maude will enforce that the inclusion A′⊆A
131

<!-- manual page 144 -->
132 CHAPTER 7. MODULE OPERATIONS
satisfies this property.
In a similar way, a system module Qspecifies a rewrite theory (Σ,E∪A,ϕ,R). A submodule
Q′ of Q will likewise specify a rewrite subtheory (Σ′,E′∪A′,ϕ′,R′) ⊆(Σ,E∪A,ϕ,R). This
means that we have inclusions Σ′ ⊆Σ, E′ ⊆E, A′ ⊆A (again, with the same equational
attributes for subsort-overloaded operators), ϕ′⊆ϕ, and R′⊆R, where ϕ′⊆ϕ is an inclusion
of functions and means that the freezing function ϕextends the function ϕ′. Note that Q′could
be a functional module, which is then understood as the rewrite theory (Σ′,E′∪A′,ϕ′
,∅), where
ϕ′specifies whatever freezing information has been given to the operators of Σ′in Q′. A system
module cannot be imported into a functional module.
In Maude, a module—any module expression giving rise to a module—can be imported as
a submodule of another in four different modes: protecting, extending, generated-by, or
including. This is done with the syntax declarations
protecting ⟨ModuleExpression⟩.
extending ⟨ModuleExpression⟩.
generated-by ⟨ModuleExpression⟩.
including ⟨ModuleExpression⟩.
which can be abbreviated, respectively, to
pr ⟨ModuleExpression⟩.
ex ⟨ModuleExpression⟩.
gb ⟨ModuleExpression⟩.
inc ⟨ModuleExpression⟩.
In addition to being allowed as arguments of a protecting, extending, generated-by, or
including importation, module expressions can also appear as the source or target of a view
(see Section 7.4.2), or as the parameter of a module, provided the top level is a theory (see
Section 7.4.3).
Each of the importation modes places specific semantic constraints on the corresponding
inclusion between the theory of the submodule and that of the supermodule. The user must
be aware that, as explained later, the Maude system does not check that these constraints are
satisfied, that is, the different modes of importation can be understood as promises by the user,
which would need to be proved by him/herself. Although those importation modes have no
effect operationally, they do crucially affect the interpretation given to a module by the theorem
proving tools. If a user is doubtful about the appropriate importation mode the default should
be to use the including mode, which places weaker requirements on the importation.
Importation statements take a module expression as argument, which may be a module
name, the summation of module expressions, the renaming of a module expression, or the
instantiation of a parameterized module expression. Modules are constructed for each subex-
pression of a module expression, and so each submodule signature must be legal. Modules and
module expressions are cached both to save time and so that the same module corresponding
to a module expression will not be imported twice via a diamond of imports. Mutually or self
recursive imports occurring through module expressions are detected and disallowed. Cached
modules generated by module expressions that no longer have any users (if the module(s) con-
taining the module expression have been replaced) are deleted. When a module M used in
a module expression is modified, any modules generated for module expressions that depend
on M are deleted and any modules that depend on M are reevaluated if you attempt to use
them. Here the notion of “depends on" is transitive through arbitrary nesting of importation
and module expressions.
In addition to being imported by the explicit importation statements we have just intro-
duced, modules can be imported in an implicit way (also in the three possible modes) by means

<!-- manual page 145 -->
7.1. MODULE IMPORTATION 133
of commands set protect/extend/generate-by/include module on/off; see more details
in Appendix A.15 and the detailed example in Section 8.1.
7.1.1 Protecting
Importing a module M′ into M in protecting mode intuitively means that no junk and no
confusion are added to M′when we include it in M. For example, we may import the module
NAT of natural numbers into a module FOO. “Junk” would be added to NAT if in FOO we have
new ground terms in canonical form in any of the sorts in NAT, namely Nat and NzNat. For
example, FOOmayhavedeclaredaconstantinfinityofsortNzNattowhichnoequationsapply.
“Confusion” would be added if different natural numbers are now identified. For example, if
FOO contains the equation s s 0 = 0, then all even numbers will be identified with 0 and all
odd numbers with s 0.
Letusexplainthesemanticsoftheprotectingrelationinmoredetailforfunctionalmodules
M′ and M, where M′ has been imported as a submodule in protecting mode, either by an
explicit protecting importation in M, or transitively through one of M’s submodules. Let
(Σ′,E′∪A′) ⊆(Σ,E∪A) be the theory inclusion defined by the module inclusion M′⊆M.
Notice that the existence of the inclusions Σ′⊆Σ, E′⊆E, and A′⊆A means that for each
sort s′in Σ′there is a well-defined function
qs′ : TΣ′/E′∪A′,s′ −→TΣ/E∪A,s′
mapping the equivalence class [t]E′∪A′ of a ground term t to the equivalence class [t]E∪A. By
definition, the submodule inclusion M′⊆M is protecting if and only if for each sort s′in Σ′the
above function is bijective. This captures mathematically the “no junk” (surjectivity) and “no
confusion” (injectivity) ideas. Under our ground Church-Rosser and termination assumptions
for M′and M this also means that the canonical form of any ground Σ′-term tin M′that has
a sort in Σ′ must be the same as its canonical form in M. The requirement that t must have
a sort is crucial. We do not require that for k′a kind the map
qk′ : TΣ′/E′∪A′,k′ −→TΣ/E∪A,k′
is bijective. The reason is that the notion of defined function—that is, an operator that disap-
pears and leaves just a term with constructors—is only meaningful when the result has a sort.
The same operator may not disappear for error terms at the kind level. That is, in the module
M extending M′ there may easily be new error terms of kind k′ created by new operators in
M. For example, if we import the module NAT into the module RAT of rational numbers, the
sorts Nat and Rat belong to the same kind, but there are now new error terms in the kind, such
as 3 + 7/0. Therefore, we should not care about “error junk” being added by a supermodule
at the kind level, provided that the sorts themselves are protected.
For system modules the protecting requirement is interpreted exactly as before as far as
their underlying equational theories are concerned. That is, if Qprotects Q′and the associated
theory inclusion is (Σ′,E′∪A′,ϕ′,R′) ⊆(Σ,E∪A,ϕ,R), then the equational theory inclusion
(Σ′,E′∪A′) ⊆(Σ,E∪A) must be protecting. We furthermore require that for any two ground
Σ′-terms t and t′ we can reach t′ from t by a sequence of rewrites in the module M′ if and
only if we can do so in the module M; that is, for ground terms in M′ we require that the
reachability relation is not altered by the supermodule.
Of course, the protecting assertion cannot be checked by Maude at runtime: it requires in-
ductive theorem proving. Using the proof techniques in [14] together with an inductive theorem
prover for membership equational logic and a Church-Rosser checker such as those described in

<!-- manual page 146 -->
134 CHAPTER 7. MODULE OPERATIONS
[43, 45, 25] (which are available in the Maude formal tool environment together with other use-
ful tools for termination and sufficient completeness), this can be done for functional modules.
Using the fact that initial models of rewrite theories are also models of equational theories [16],
similar proof techniques could be developed to prove the protecting relation between rewrite
theories.
7.1.2 Extending
A weaker, yet substantial, requirement about a module importation is expressed by the key-
word extending. Intuitively, the idea is to allow “junk,” but to rule out confusion. Extending
importations may appear naturally in situations in which the data of some sort is extended
with new data elements, yet not identifying previously defined data, like adding a new constant
infinity to the natural numbers in a module importing NAT. As another example, when defin-
ing the semantics of a programming language in Maude, we can have from the beginning a sort
Program, and define incrementally the syntax of programs in several modules, say, EXPRESSION,
STATEMENT, PROCEDURE, and so on. This will typically give rise to a family of extendingmodule
importations as more syntax is added.
For functional modules M′ and M, where M′ has been imported as a submodule in
extending mode, either by an explicit extending importation in M, or transitively through
one of M’s submodules, if (Σ′,E′∪A′) ⊆(Σ,E ∪A) is the theory inclusion defined by the
module inclusion M′ ⊆M, the extending requirement means that for each sort s′ in Σ′ the
function
qs′ : TΣ′/E′∪A′,s′ −→TΣ/E∪A,s′
is injective. For system modules the extending requirement is interpreted exactly as before
as far as their underlying equational theories are concerned. That is, if Q extends Q′ and the
associated theory inclusion is (Σ′,E′∪A′,ϕ′,R′) ⊆(Σ,E∪A,ϕ,R), then the equational theory
inclusion (Σ′,E′∪A′) ⊆(Σ,E∪A) must be extending. We furthermore require that for any
two ground Σ′-terms tand t′we can reach t′from tby a sequence of rewrites in the module M′
if and only if we can do so in the module M; that is, for ground terms in M′ the reachability
relation is not altered by the supermodule.
Under the Church-Rosser and termination assumptions, the extending (Σ′,E′∪A′) ⊆
(Σ,E ∪A) requirement is a form of conservative extension requirement, in the sense that it
implies that for any Σ′ground terms t and t′that have a sort in (Σ′,E′∪A′), E′∪A′proves
t= t′if and only if E∪Aproves t= t′. In addition, for system modules it further implies that
for any two ground Σ′-terms t and t′ the reachability relation is not altered by the extension.
In summary, equality and reachability are conservatively preserved for ground terms.
Note that the extendingrelation does not destroy protecting importations further down the
hierarchy. That is, if M imports M′in extending mode, but M′imports M′′in protecting
mode, then M still imports M′′ in protecting mode, not in extending mode. If we do not
want M to protect M′′ (because this is indeed violated), then we have to say so by explicitly
giving an extending importation declaration for M′′in M.
7.1.3 Generated-by
The protecting keyword requires that submodules are imported with “no junk and no con-
fusion,” whereas the extending keyword just requires that submodules are imported with
“no confusion.” The generated-by keyword covers the other alternative weakening of the
protecting requirement, namely, requiring that submodules are imported with “no junk.” A
simple example is the importation of the NAT module into the module:

<!-- manual page 147 -->
7.1. MODULE IMPORTATION 135
fmod NAT/3 is
generated-by NAT .
eq s(s(s(0)) = 0 .
endfm
which does not add any “junk” to the module NAT, i.e., does not add any new data elements to
its Nat sort, although of course it adds considerable “confusion,” since now 3 = 0.
The semantics of the generated-bykeyword is then easy to explain. For functional modules
M′ and M, where M′ has been imported as a submodule in generated-by mode, either by
an explicit generated-by importation in M, or transitively through one of M’s submodules, if
(Σ′,E′∪A′) ⊆(Σ,E∪A) is the theory inclusion defined by the module inclusion M′⊆M, the
generated-by requirement means that for each sort s′in Σ′the function
qs′ : TΣ′/E′∪A′,s′ →TΣ/E∪A,s′
issurjective. Forsystemmodulesthegenerated-byrequirementisinterpretedexactlyasbefore
as far as their underlying equational theories are concerned. That is, if Q extends Q′ and the
associated theory inclusion is (Σ′,E′∪A′,ϕ′,R′) ⊆(Σ,E∪A,ϕ,R), then the equational theory
inclusion (Σ′,E′∪A′) ⊆(Σ,E∪A) must satisfy the generated-by requirement.
7.1.4 Including
The most general form of module importation is provided by the including keyword. No
requirements are made in an including importation about maps of the form qs′ : there can now
bejunk(lackofsurjectivity)and/orconfusion(lackofinjectivity). Likewise, forsystemmodules
it is not anymore required that the reachability relation between ground terms in the submodule
is preserved. The including keyword does however impose some requirements. First of all,
there is the requirement that the equational attributes of subsort-overloaded operators must
be the same. Furthermore, the including relation does not destroy protecting or extending
importations further down the hierarchy. That is, if M imports M′in includingmode, but M′
imports M′′in protecting (resp. extending) mode, then M still imports M′′in protecting
(resp. extending) mode, not in including mode. If we do not want M to protect or extend
M′′(because this is indeed violated), then we have to say so by explicitly giving an including
importation declaration for M′′in M.
Given that, as already mentioned, there is no difference at runtime between the different
modes of importation because the Maude system does not check the corresponding require-
ments, from a pragmatic point of view, when a user is doubtful about the appropriate impor-
tation mode, the best idea is to use the including mode so that at least no false assertion is
made.
7.1.5 Default conventions in module importations
We have already explained our default convention when a submodule M0 is imported indirectly
and transitively into M through the direct importation by M of a module M1 that itself imports
M0. Then, whatever was the mode (protecting, extending, generated-by, or including)
in which M0 was imported by M1 is also, by default, the mode in which M0 is imported by M,
unless M contains an explicit declaration importing M0 in a different mode. We now explain
what our default convention is in the case of diamond importations.
We talk of a diamond importation of M0 by M, when M0 is imported indirectly by M
through the direct importation of two or more different modules, say M1,M2,...,Mk. The
problem now is that M0 can be imported by each of the modules M1,M2,...,Mk in different

<!-- manual page 148 -->
136 CHAPTER 7. MODULE OPERATIONS
modes. For example, M1 could import it in protecting mode, M2 in extending mode, M3 in
including mode, and so on. What should now be the default convention for the mode in which
M imports M0? We adopt a convention that is consistent with a logical understanding of such
importation declarations. Indeed, such declarations impose semantic constraints of decreasing
strength; that is, we have:
including M0
08
fn
extending M0
generated-by M0
08fn
protecting M0
The default convention consistent with this logical reading is that the strongest mode wins,
i.e., protecting prevails over extending and generated-by, which themselves prevail over
including. This is because we view the set of all such importing mode declarations as a
conjunction, and exploit the logical equivalence between A ⇒ B and (A∧B) ⇔ A. The
importationofamoduleinbothgenerated-byandextendingmodesisequivalenttoimporting
it in protecting mode.
Note that this “strongest wins” default mode may not always be the correct or intended
mode in which M should import M0. Sometimes it may not be, and then the user should
overrule the default convention by declaring explicitly a different mode in which M imports
M0. A pragmatic reason why this need for overruling the default mode may arise is that,
although a weaker mode of importation (say extending) does not logically preclude such an
importation also satisfying a stronger one (say protecting), in practice, when we declare an
importation in a weaker mode it can often be because we know or suspect that it fails to satisfy
a stronger mode. For example, when we say “extending” we may often mean “extending and
not protecting.”
7.1.6 Some module hierarchy examples
Prime numbers sieve
Section 4.4.7 included a functional module specifying the sieve of Eratosthenes to calculate
prime numbers.
fmod SIEVE is
protecting NAT .
sort NatList .
subsort Nat < NatList .
...
endfm
The predefined module of natural numbers (see Section 8.3) is imported in protecting
mode. This is justified because the elements of sort Nat are used to generate the lists of natural
numbers by means of a subsort declaration and also as arguments of other operators. However,
no new operator of result sort Nat is added in the SIEVE module, and all the equations in this
module identify elements of sort NatList without identifying different natural numbers.
Vending machine
The vending machine example in Section 5.1 was presented in a modular way, by separating the
underlyingsignaturedefiningthestatesofthemachinefromtherulesdefiningthecorresponding
transitions.

<!-- manual page 149 -->
7.1. MODULE IMPORTATION 137
mod VENDING-MACHINE is
including VENDING-MACHINE-SIGNATURE .
var M : Marking .
rl [add-q] : M => M q .
...
endm
It is important to notice that in this example the importation mode cannot be either
protecting or extending, because those modes require preservation of the reachability re-
lation, which clearly is not the case when adding (non-identity) rewrite rules to a functional
module (where the reachability relation is the identity).
Bank accounts and object configurations
In Section 6.1.4, we presented an object-oriented module ACCOUNT specifying bank accounts.
In order to run some tests, the following module introduces three new constants to be used as
object identifiers, and a new constant to be used as a test configuration. This configuration
constant is identified with a term of sort Configuration in the imported module ACCOUNT
by means of an equation whose righthand side is omitted below. However, constants A-001,
A-002, and A-003 are new data elements, i.e., junk, of sort Oid. Sorts Oid and Configuration
are declared in the predefined module CONFIGURATION (see Section 6.1), which is implicitly
imported in including mode in all object-oriented modules. Since imported in including
mode in ACCOUNT, and also in ACCOUNT-TEST, it is not necessary to import it in a different
mode. However, since junk is being added, the ACCOUNT module should be imported in the
ACCOUNT-TEST module in either including or extending modes. Therefore, the appropriate
importation mode is extending.
omod ACCOUNT-TEST is
ex ACCOUNT .
ops A-001 A-002 A-003 : -> Oid .
op bankConf : -> Configuration .
eq bankConf = ... .
endom
The following example, from Section 7.5, is more interesting, in that it introduces new sorts
MsgBody and Request, not just new constants for a sort in the imported module. Still, the
appropriate importation mode is extending because there are no new rewrite rules and no
equations, and thus no confusion between elements in imported sorts is introduced.
mod DATA-AGENTS-CONF is
ex CONFIGURATION .
sort MsgBody .
op msg : Oid Oid MsgBody -> Msg [ctor message] .
sort Request .
op w4 : Oid Oid MsgBody -> Request [ctor] .
endm
There are several other modules in Chapter 6.4 illustrating the use of the extending mode
in importing modules, like BANK-MANAGER-TEST, TICKER-TEST, TICKER-FACTORY-TEST, and
AGENT-TEST; see Figures 6.1, 6.2, and 7.3.
Hierarchy of predefined modules
A more complex acyclic importation graph corresponds to the hierarchy of predefined modules
for basic data types, described later in Chapter 8 and shown in Figure 8.1, where all the
importations are in protecting mode.

<!-- manual page 150 -->
138 CHAPTER 7. MODULE OPERATIONS
7.2 The summation module expression
The summation module operation creates a new module that includes all the information in its
summands. The syntax for a summation of module expressions is
⟨ModuleExpression⟩ + ⟨ModuleExpression⟩
with + associative and commutative.
Summation expressions are flattened before being evaluated, so that A + (B + C) and
(C + A) + B both create a single new module A + B + C. The evaluation of a summation
module expression results in the creation of a new module, with such a module expression as
its name, which imports the module expressions being combined. The new module will be gen-
erated having one type or another, depending on the types of the arguments of the summation
module expression. A summation is a functional module if all the summands are functional
modules and a system module otherwise.
Although the use of the summation module expression is more interesting in combination
with other module expressions, let us consider as an example the following module, in which
the union of the predefined FLOAT and STRING modules (see Chapter 8) are imported together
in protecting mode to illustrate its use.
fmod FLOAT-STRING is
protecting FLOAT + STRING .
...
endfm
Notice that a declaration
protecting A + B .
is not equivalent to a sequence of declarations
protecting A .
protecting B .
because in general the modules A and B may share sorts and operators. The same happens with
extending declarations, for the same reason. However, a declaration of the form
including A + B .
is indeed equivalent to a sequence of declarations
including A .
including B .
7.3 Module renaming
The syntax of a renaming module expression is
⟨ModuleExpression⟩ * ( ⟨Renaming⟩ )
where ⟨Renaming⟩is a comma-separated sequence of renaming items of the forms:
sort ⟨identifier⟩ to ⟨identifier⟩
op ⟨identifier⟩ to ⟨identifier⟩
op ⟨identifier⟩ to ⟨identifier⟩ [ ⟨attribute-set⟩ ]
op ⟨identifier⟩ : ⟨type-list⟩ -> ⟨type⟩ to ⟨identifier⟩
op ⟨identifier⟩ : ⟨type-list⟩ -> ⟨type⟩ to ⟨identifier⟩ [ ⟨attribute-set⟩ ]
label ⟨identifier⟩ to ⟨identifier⟩

<!-- manual page 151 -->
7.3. MODULE RENAMING 139
Note that, in addition to the typical renamings of sorts and operators, renaming of labels is
also supported (which may be useful for metalevel applications). For object-oriented modules
additional renamings also are available:
class ⟨identifier⟩ to ⟨identifier⟩
attr ⟨attr-identifier⟩ to ⟨attr-identifier⟩
msg ⟨identifier⟩ to ⟨identifier⟩
msg ⟨identifier⟩ to ⟨identifier⟩ [ ⟨attribute-set⟩ ]
msg ⟨identifier⟩ : ⟨type-list⟩ -> ⟨type⟩ to ⟨identifier⟩
msg ⟨identifier⟩ : ⟨type-list⟩ -> ⟨type⟩ to ⟨identifier⟩ [ ⟨attribute-set⟩ ]
Renaming (_*(_)) binds tighter than summation (_+_), and it groups to the left. Note
how the renaming of operators (and message constructors) allows changing the attributes of
the operator being renamed. The only attributes currently allowed in operator maps are prec,
gather, andformat. Theideaisthatwhenyourenameanoperator, theoldsyntacticproperties
may no longer be legal and are reset to defaults, unless you explicitly set them with these
attributes; for example, when a change in the syntax of the operator could cause a parsing
different from the intended one. Let us see an example in which modifying the grammatical
attributes of an operator is useful. Consider the following module defining lists of natural
numbers with a max operator returning the greatest of the elements in a list of natural numbers.
fmod NAT-LIST-MAX is
pr NAT .
sort NeNatList .
subsort Nat < NeNatList .
op __ : NeNatList NeNatList -> NeNatList [ctor assoc] .
op max : NeNatList -> Nat .
var N : Nat .
var NL : NeNatList .
eq max(N) = N .
eq max(N NL) = if N > max(NL) then N else max(NL) fi .
endfm
We may obtain the maximum of a list of natural numbers as follows.
Maude> red max(4 2 5 3) .
result NzNat: 5
Suppose now that we want to change the syntax of the function max in the NAT-LIST-MAX
module above to maximum_.
fmod NAIVE-NAT-LIST-MIXFIX-MAX is
pr NAT-LIST-MAX * (op max : NeNatList -> Nat to maximum_) .
endfm
We can do the following reduction:
Maude> red maximum 2 3 4 1 .
result NeNatList: 2 3 4 1
This result may seem strange, but it makes perfect sense. What has happened is that it has
been parsed as (maximum 2) 3 4 1, the only possible parse given the default precedence values
and gathering patterns assigned. Since by default maximum_ has precedence 15 and gathering E,
it cannot take the list 2 3 4 1 as argument because__ has precedence 41. However, since__
has gathering e E, maximum 2is a valid argument for it (see Section 3.9 for a detailed discussion
on the use of precedence values and gathering patterns and their default values). We can of
course obtain the intended result by placing parentheses around the set of numbers,

<!-- manual page 152 -->
140 CHAPTER 7. MODULE OPERATIONS
Maude> red maximum (2 3 4 1) .
result NzNat: 4
butit ismoreconvenientto changethe precedencevaluesoftheoperators. We can, forexample,
raise the precedence of maximum_.
fmod NAT-LIST-MIXFIX-MAX is
pr NAT-LIST-MAX
* (op max : NeNatList -> Nat to maximum_ [prec 41]) .
endfm
having then the following reduction.
Maude> red maximum 2 3 4 1 .
result NzNat: 4
Notice that if maximum_ has precedence 41, then (maximum 2) 3 4 1 is no longer a valid parse.
A renaming can be considered as a function that, given a module M and a list of mappings
S, returns a copy of the module, with such a module expression as its name, in which the
names of sorts, operators, etc. are changed as indicated by the mappings. However, renaming
a module that has imports is a subtle issue. Given a structured specification, the renaming not
only causes the creation of a copy of the top module in the structure, but renames also the part
of the submodule structure that is affected by the renaming. For any other submodule M′ in
the structure which is affected by the mappings, a renamed copy of it is generated with name
M′ * (S′), where S′is the subset of mappings in S that affect M′
.
A module expression A * (R) evaluates to A if A has no content that is affected by the
renaming R. Otherwise A * (R) evaluates to a new module A * (R′) where R′ is obtained
by deleting those renaming items that do not affect A, and canonizing the types in operator
renamings with respect to A(see below). If Aimports modules Band C, A * (R′) will import
modules obtained by evaluating B * (R′) and C * (R′).
There are some subtle cases. Consider for example the following three modules:
fmod RENAMING-EX-A is
sort Foo .
op a : -> Foo .
op f : Foo -> Foo .
endfm
fmod RENAMING-EX-B is
including RENAMING-EX-A .
sort Bar .
subsort Foo < Bar .
op f : Bar -> Bar .
endfm
fmod RENAMING-EX-C is
inc RENAMING-EX-B * (op f : Bar -> Bar to g) .
endfm
Here, the operator f in the module RENAMING-EX-A looks as though it is not affected by the
renaming in the module RENAMING-EX-C, but because of the subsort declaration Foo < Bar in
RENAMING-EX-B, it should be renamed for consistency. This is internally handled by the Maude
system by canonizing the type Bar occurring in the renaming
op f : Bar -> Bar to g

<!-- manual page 153 -->
7.4. PARAMETERIZED PROGRAMMING 141
to the kind expression [Foo,Bar], which includes all of the sorts in the kind [Bar] occurring
in RENAMING-EX-B. Thus, the module expression
RENAMING-EX-B * (op f : Bar -> Bar to g)
evaluates to a new module
RENAMING-EX-B * (op f : [Foo,Bar] -> [Foo,Bar] to g)
which includes the module expression
RENAMING-EX-A * (op f : [Foo,Bar] -> [Foo,Bar] to g)
which evaluates to a new module
RENAMING-EX-A * (op f : [Foo] -> [Foo] to g)
in which f has been renamed.
In general, * does not distribute over +. Consider this other example:
fmod RENAMING-EX-D is
sorts Foo Bar .
endfm
fmod RENAMING-EX-E is
inc RENAMING-EX-D .
op f : Foo -> Foo .
endfm
fmod RENAMING-EX-F is
inc RENAMING-EX-D .
subsort Foo < Bar .
op f : Bar -> Bar .
endfm
and
It is not the case that the module expressions
(RENAMING-EX-E + RENAMING-EX-F) * (op f : Bar -> Bar to g)
(RENAMING-EX-E * (op f : Bar -> Bar to g))
+ (RENAMING-EX-F * (op f : Bar -> Bar to g))
evaluate to the same module, because in the latter the operator f occurring in RENAMING-EX-E
will not be renamed, since f : Bar -> Bar does not occur in RENAMING-EX-E.
Operators with the poly attribute are only affected by operator renamings that do not
specify types. Renaming a module does not change its module type, that is, renamed functional
modules (resp. system modules) remain functional (resp. system).
7.4 Parameterized programming
Theories, parameterized modules, and views are the basic building blocks of parameterized
programming [17, 69]. As in OBJ, a theory defines the interface of a parameterized module,
that is, the structure and properties required of an actual parameter.
A parameterized module is a module with one or more parameters, each of which is expressed
by means of one theory, that is, modules can be parameterized by one or more theories. If we
want,e.g.,todefinealistorasetofelements,wemaydefineamoduleLISTorSETparameterized

<!-- manual page 154 -->
142 CHAPTER 7. MODULE OPERATIONS
by a theory expressing the requirements on the type of the elements to store in such data
structures. Thus, theories are used to declare the interface requirements for parameterized
modules. In the case of lists and sets we do not need any requirement on the data elements,
and therefore we may use the trivial theory TRIV, with just a sort Elt, as parameter of such
modules; but in other cases, say search trees or sorted lists, we may require, e.g., a particular
operator, an order relation, or an equivalence relation, in which cases we shall need to use the
appropriate theories describing the specific requirements.
Theinstantiationoftheformalparametersofaparameterizedmodulewithactualparameter
modules or theories requires a view mapping entities from the formal interface theory to the
correspondingentitiesintheactualparametermodule. Viewscanalsobeparameterized, which,
as we can see in Section 7.4.7, may greatly improve reusability of specifications.
7.4.1 Theories
Theories are used to declare module interfaces, namely the syntactic and semantic properties to
be satisfied by the actual parameter modules used in an instantiation. As for modules, Maude
supports three different types of theories: functional theories, system theories, and object-
oriented theories, with the same structure of their module counterparts, but with a different
semantics. Functional theories are declared with the keywords fth ... endfth, system theo-
ries with the keywords th ... endth, and object-oriented theories with oth ... endoth.The
three of them can have sorts, subsort relationships, operators, variables, membership axioms,
and equations, and can import other theories or modules. System theories can also have rules.
Object-oriented theories may include as well declarations of classes, inheritance relationships,
and messages. Although there is no restriction on the operator attributes that can be used in
a theory, there are some subtle restrictions and issues regarding the mapping of such operators
(see Section 7.4.2).
Like functional modules, functional theories are membership equational logic theories, but
they do not need to be Church-Rosser and terminating, and therefore some or all of their
statements may be declared with the nonexec attribute. Theories have a loose semantics,
in the sense that any algebra satisfying the equations and membership axioms in the theory
is an acceptable model. However, functional theories can be executed in exactly the same
way as functional modules; that is, the equations and membership axioms not having the
nonexecattributeshouldbeChurch-Rosserandterminating, andcanbeexecutedbyequational
simplification, whereas the statements declared as nonexec can be arbitrary and can only be
executed in a controlled way at the metalevel. System and object-oriented theories have a
similar loose interpretation, but are treated, respectively, just as system or object-oriented
modules for executability purposes. Theories are then allowed to contain rules and equations
which, if declared with the nonexec attribute, can be arbitrary, that is, can have variables in
their righthand sides or conditions that may not appear in their corresponding lefthand sides
and do not obey the admissibility conditions in Sections 4.6 and 5.3. Similarly, conditional
membership axioms may have variables in their conditions that do not appear in their head
membership assertions. Also, the lefthand side may be a single variable.
Constants in theories may be given a pconstattribute. It marks the constant as a parameter
constant. For example:
op c : -> Foo [pconst] .
This attribute is mostly inert, but when a constant c with this attribute is declared in a theory
T, and a module M has a parameter X :: T, M receives a constant X$c rather than c. The
pconst attribute cannot be combined with the poly attribute. Constants with the pconst

<!-- manual page 155 -->
7.4. PARAMETERIZED PROGRAMMING 143
attribute cannot be used on either side of an operator to term mapping in a view. See details
on parameter constants in Section 7.4.3.
Let us begin by introducing the functional theory TRIV, which requires just a sort.
fth TRIV is
sort Elt .
endfth
The theory TRIV is used very often, for instance in the definition of data structures, such as
lists, sets, trees, etc., of elements of some type with no specific requirement; in these cases, it
is common to define a module, say LIST, SET, TREE, etc., parameterized by the TRIV theory
(see Section 7.4.3). The theory TRIV is predefined in Maude, together with several useful views
from TRIV to other predefined modules and theories (see Section 8.13.1).
But we can define more interesting theories. For example, the theory of monoids, with an
associative binary operator with identity element 1, can be specified as follows:
fth MONOID is
including TRIV .
op 1 : -> Elt .
op __ : Elt Elt -> Elt [assoc id: 1] .
endfth
Notice the importation of the theory TRIV into the MONOID theory. As for modules, it is
possible to structure our theories by importing other theories and modules (and in general
module expressions involving theories and modules) into theories. However, a theory cannot
be imported into a module: theories can only be used as parameters of modules. Also, theories
do not have automatic importation as modules do (e.g., BOOL, as described in Section 8.1).
Modules and theories can be combined in module expressions (they can be summed, for
example), and modules can be imported into theories. Basically, we have a lattice
system theory
functional theory system module
functional module
HHH
HH
H
where summation corresponds to join, and where a module or theory may only import a sub-
module or subtheory of lesser or equal type.
Although the importation of a module into a theory can be done in any mode, a theory can
only be imported in including mode into another theory. The including importation of a
theory into another theory keeps its loose semantics. However, the importation of a theory into
anotheroneinprotecting,generated-byorextendingmodewouldimplyadditionalsemantic
requirements; such modes of importation are ruled out.1 On the other hand, although a module
keeps its initial interpretation when imported into a theory in protecting, generated-by or
extending modes, it looses it if imported in including mode.
Let us see a few examples illustrating all this.
The theory of commutative monoids can be defined just as the theory of monoids; the binary
operator _+_ is now declared associative, commutative, and has 0 as its identity element.
1If a theory is imported using a mode other than including, the system gives an error message saying that
the mode is being treated as if it were including. Other illegal importations give an error message saying that
they are being ignored.

<!-- manual page 156 -->
144 CHAPTER 7. MODULE OPERATIONS
fth +MONOID is
including TRIV .
op 0 : -> Elt .
op _+_ : Elt Elt -> Elt [assoc comm id: 0] .
endfth
The theory of semirings can be expressed as follows.
fth SEMIRING is
including MONOID .
including +MONOID .
vars X Y Z : Elt .
eq X (Y + Z) = (X Y) + (X Z) [nonexec] .
eq (X + Y) Z = (X Z) + (Y Z) [nonexec] .
endfth
Note the use of the nonexec attribute, and note also that given the semantics of theory
inclusions, there is no difference between having a structured theory or one flat theory including
all the declarations.2 For example, the theory of commutative rings can be defined directly as
follows:
fth RING is
sort Ring .
ops z e : -> Ring .
op _+_ : Ring Ring -> Ring [assoc comm id: z] .
op _*_ : Ring Ring -> Ring [assoc comm id: e] .
op -_ : Ring -> Ring .
vars A B C : Ring .
eq A + (- A) = z [nonexec] .
eq A * (B + C) = (A * B) + (A * C) [nonexec] .
endfth
but could also be defined as a structured theory including the theories of commutative groups
and commutative monoids (renamed if necessary), to which the distributivity axiom is added.
As mentioned above, the including importation of a theory into another theory keeps its
loose semantics. However, if the imported theory contains a module, which therefore must be
interpreted with an initial semantics (see Section 5.3), then that initial semantics is maintained
by the importation. For example, in the definition of the TAOSET theory below, the declaration
protecting BOOL ensures that the initial semantics of the functional module for the Booleans
is preserved, which is in fact a crucial requirement.
Let us consider now a hierarchy of theories for partially and totally ordered sets. The most
basic theory specifies a transitive and antisymmetric order _<_ on a set:
fth TAOSET is
protecting BOOL .
sort Elt .
op _<_ : Elt Elt -> Bool .
vars X Y Z : Elt .
ceq X < Z = true if X < Y /\ Y < Z [nonexec label transitive] .
ceq X = Y if X < Y /\ Y < X [nonexec label antisymmetric] .
endfth
2The only exception to this semantic equivalence between structured theories and their flattened form is the
case in which a theory imports some modules, since any of the protecting, generated-by or extending initiality
requirements of the imported module and its submodules must be preserved. Those requirements would be lost
if the whole structure were to be flattened.

<!-- manual page 157 -->
7.4. PARAMETERIZED PROGRAMMING 145
By adding irreflexivity to TAOSET we get a theory specifying a strict partial order:
fth SPOSET is
including TAOSET .
var X : Elt .
eq X < X = false [nonexec label irreflexive] .
endfth
Notice that in this case antisymmetry is implied by irreflexivity and transitivity. Of course,
there are different ways of presenting a theory, and in particular one can always write the corre-
sponding flat theory with only the axioms for irreflexivity and transitivity. In the presentation
above, the initial semantics of BOOL when it is imported in protecting mode in TAOSET is
preserved when the latter is included in SPOSET. The same will hold in further importations in
this hierarchy of order theories.
On the other hand, by adding reflexivity to TAOSET we get a theory specifying a non-strict
partial order. Notice the renaming in the importation, so that the name of the order _<=_
reflects its reflexivity.
fth NSPOSET is
including TAOSET * (op _<_ to _<=_) .
var X : Elt .
eq X <= X = true [nonexec label reflexive] .
endfth
Having both _<_ and _<=_ available together is useful in some applications. There are
standard ways of associating a strict partial order with a non-strict partial order and the other
way around:
• from a<b, one can define a≤b as equivalent to a<b or a= b; and
• from a≤b, one can define a<b as equivalent to a≤b and a̸= b.
These equivalences can be expressed as Maude theories as follows, where we use the same name
for both theories because they are equivalent, that is, we have two different presentations of
the same theory and in what follows we will not care about which version of POSET is used.
fth POSET is
including SPOSET .
op _<=_ : Elt Elt -> Bool .
vars X Y : Elt .
eq X <= X = true [nonexec] .
ceq X <= Y = true if X < Y [nonexec] .
ceq X = Y if X <= Y /\ X < Y = false [nonexec] .
endfth
fth POSET is
including NSPOSET .
op _<_ : Elt Elt -> Bool .
vars X Y : Elt .
eq X < X = false [nonexec] .
ceq X <= Y = true if X < Y [nonexec] .
ceq X = Y if X <= Y /\ X < Y = false [nonexec] .
endfth
Notice that the axioms are almost the same in both presentations of POSET, but, while the
first presentation defines the reflexive order _<=_ in terms of the irreflexive one _<_, the second
presentation defines the irreflexive order _<_ in terms of the reflexive one _<=_.

<!-- manual page 158 -->
146 CHAPTER 7. MODULE OPERATIONS
To each of the previous theories we can add an axiom requiring the order to be total (or
linear), that is, two different elements have to be related one way or the other. In this way, we
have the following theories for a strict total order, a non-strict total order, and a total order
with both operations.
fth STOSET is
including SPOSET .
vars X Y : Elt .
ceq X = Y if X < Y = false /\ Y < X = false [nonexec label total] .
endfth
fth NSTOSET is
including NSPOSET .
vars X Y : Elt .
ceq X <= Y = true if Y <= X = false [nonexec label total] .
endfth
fth TOSET is
including POSET .
vars X Y : Elt .
ceq X <= Y = true if Y <= X = false [nonexec label total] .
endfth
As already mentioned above, the requirement ensuring the initial semantics of BOOL when it
is protected in TAOSET is then preserved by the remaining theories when TAOSET is included in
them via a chain of including importations. In fact, we are dealing with structures in which
part of them, not only the top theory, has a loose semantics, while other parts contain modules
with an initial semantics.
This hierarchy of order theories is displayed in Figure 7.1, where we represent by boxes
the modules (with initiality constraints), by ovals the theories (with loose semantics), by triple
arrows the protecting importations, and by single arrows the including importations.
Finally, as an example of a system theory, let us consider the theory CHOICE of bags of
elements with a choice operator defined on the bags by a rewrite rule that nondeterministically
picks up one of the elements in the bag. We can specify this theory as follows, where we have
a sort Bag declared as a supersort of the sort Elt.
th CHOICE is
sorts Bag Elt .
subsort Elt < Bag .
op empty : -> Bag .
op __ : Bag Bag -> Bag [assoc comm id: empty] .
op choice : Bag -> Elt .
var E : Elt .
var B : Bag .
rl [choice] : choice(E B) => E .
endth
Let us conclude this section with a word on object-oriented theories. As already explained,
their structure is the same as that of object-oriented modules. Object-oriented theories can
have classes, subclass relationships, and messages. These object-oriented notions may be useful
for the definition of theories; for example, the following theory CELL specifies the theory of
classes with at least one attribute of any sort.
oth CELL is
sort Elt .

<!-- manual page 159 -->
7.4. PARAMETERIZED PROGRAMMING 147
TOSET
STOSET POSET
NSTOSET
SPOSET
NSPOSET
TAOSET
BOOL
Figure 7.1: Hierarchy of order theories
class Cell | contents : Elt .
endoth
7.4.2 Views
In Maude, views are used to specify how a particular target module or theory is claimed to
satisfy a source theory. In general, there may be several ways in which such requirements might
be satisfied, if at all, by the target module or theory; that is, there can be different views, each
specifying a particular interpretation of the source theory in the target. Each view declaration
has an associated set of proof obligations, namely, for each axiom in the source theory it should
be the case that the axiom’s translation by the view holds true in the target. Since the target
can be a module interpreted initially, verifying such proof obligations may in general require
inductive proof techniques of the style supported for Maude’s logic in [25], and for which tools
in the Maude formal environment can be used. Such proof obligations are not discharged or
checked by the system.
In the definition of a view we have to indicate its name, the source theory, the target module
or theory, and the mapping of each sort and operator in the source theory. The name space
of views is separate from the name space of modules and theories, which means that, e.g., a
view and a module could have the same name. In fact, we shall see below how we recommend
naming inclusion views as the target theory. The source and target of a view can be any module
expression, with the source module expression evaluating to a theory and the target module
expression evaluating to a module or a theory.
The syntax for views is as follows:
view ⟨ViewName⟩ from ⟨Source⟩ to ⟨Target⟩ is
⟨Mappings⟩
endv

<!-- manual page 160 -->
148 CHAPTER 7. MODULE OPERATIONS
View names are simple identifiers (a single token), as defined in Section 3.1, but they cannot
contain neither underscores nor escaped parentheses or commas (‘_’, ‘(’, ‘)’, ‘[’, ‘]’, ‘’, ‘’, ‘,’). As
we will see in Section 7.4.4, when a parameterized constant c{X}, is instantiated with a view
V, it becomes c{V}. Since constant operator names cannot contain underscores, this restriction
must also be applied to views. Moving up and down the metalevel may cause problems with
parentheses and commas. These restrictions are enforced during the analysis phase after surface
parsing.
The mapping of a sort in the source theory to a sort in the target module or theory is
expressed with syntax
sort ⟨identifier⟩ to ⟨identifier⟩.
For each sort S in the source theory, there must exist a sort S′in the target module or theory
whichisitsmappingundertheview; unmentionedsortsgettheidentitymapping. Furthermore,
if sorts S and T in the source theory are in the same kind, then their mappings S′and T′under
the view must be in the same kind in the target module or theory. Finally, if S is a subsort of
T, then it must be true that S′is a subsort of T′
.
The mapping of operators is expressed with syntax
op ⟨identifier⟩ to ⟨identifier⟩.
op ⟨identifier⟩ : ⟨type-list⟩ -> ⟨type⟩ to ⟨identifier⟩.
op ⟨op-expr⟩ to term ⟨term⟩.
In the first case, where only an operator identifier is given, the map affects all operators with
the same name. Existence of appropriate operators in the target is checked for. In the second
case, when explicit arity and coarity are given, the operator map affects not only the operators
with such arity and coarity, but also the entire family of subsort-overloaded operators (see
Section 3.6) associated with the given operator. The third case is similar to the second one,
but instead of mapping the operator to another operator, it is mapped to a given term with
variables; ⟨op-expr⟩is a term consisting of a single operator applied to variables—declared
either on-the-fly or with variable declarations in the same view—and the target term is any
term with variables, those in the source ⟨op-expr⟩in the corresponding sorts resulting from the
mapping. See below for more details and examples.
Viewswithobject-orientedtheoriesassourcesandobject-orientedmodulesorobject-oriented
theoriesastargetscanalsobedefined. Foraviewhavinganobject-orientedtheoryasitssource,
the mapping of a class in the source theory to a class in the target is expressed with syntax
class ⟨identifier⟩ to ⟨identifier⟩.
Attribute maps have the form
attr ⟨identifier⟩ to ⟨identifier⟩.
where the first identifier is the name of an attribute in the source theory and the second one is
an attribute in the target module or theory. Note that there might be homonymous attributes
in different classes.
The mapping of message constructors is expressed with syntax
msg ⟨identifier⟩ to ⟨identifier⟩.
msg ⟨identifier⟩ : ⟨type-list⟩ -> ⟨type⟩ to ⟨identifier⟩.
where a message identifier or a message identifier together with its arity and value sort is
mapped to another one in the target module or theory. As for operators, a message operator
map in which explicit arity and coarity are given affects the entire family of subsort-overloaded
message operator declarations associated with the declaration of the given message.

<!-- manual page 161 -->
7.4. PARAMETERIZED PROGRAMMING 149
Maps must preserve the arities and the types of operators and message constructors, and
sort maps, and operator and message constructor maps must be compatible. For each operator
f : S1 ...Sn -> T in the source theory there must exist an operator f′: S′
1 ...S′
n -> T′in the
target module or theory, where S′
i is the mapping of sort Si under such a view.
Unmentioned operators and message constructors also get the identity mapping. Thus,
“obvious” parts of a mapping do not need to be explicitly given, namely, any identical mapping
of a sort or class, or operator or message constructor such that its arity and coarity are mapped
to thoseof anoperatoror message constructorwith thesame namein the targetcan be omitted.
As a first example, the following view StringAsToset defines a view from the theory TOSET,
presented in Section 7.4.1, to the predefined functional module STRING, described in Section 8.9.
view StringAsToset from TOSET to STRING is
sort Elt to String .
endv
Notice that the identity maps op _<_ to _<_ and op _<=_ to _<=_ have been omitted.
The maps sending operators to derived operators, that is, terms with variables, allow us to
map an operator, not only to another operator, but also to an expression. The view RingToRat
below is a view from the theory RING, presented in Section 7.4.1, to the predefined functional
module RAT, described in Section 8.7.
view RingToRat from RING to RAT is
sort Ring to Rat .
op e to term 1 .
op z to 0 .
endv
Notice that we have followed the convention of omitting the “obvious” parts of the map con-
cerning the operators _+_ and _*_. Furthermore, we have used an operator map sending the
operator e to the term 1, due to the fact that in RAT 1 is not a constant, but the term s_^1(0)
(see Sections 4.4.2, 8.3, and 8.7 for details). Note that the map op e to term 1 cannot be ex-
pressed with the other forms of operator maps, because 1 is not an operator, but just syntactic
sugar for the term s_^1(0).
As another example, consider the view from the theory NSPOSET, in which we have a
sort Elt and a non-strict “less or equal” operator _<=_ : Elt Elt -> Bool, to a module
defining the integers with no such operator but instead with a strict operator “less than”
_<_ : Int Int -> Bool. Then, we can define a view with maps
sort Elt to Int .
op X:Elt <= Y:Elt to term X:Int < Y:Int or X:Int == Y:Int .
where we have also used the predefined equality operator _==_. The lefthand side of the oper-
ator mapping, X:Elt <= Y:Elt in this case, which consists of an operator with only variable
arguments, must parse to a unique term in the source theory. Each of the variables used in the
maps must have a unique base name (e.g., using both X:Foo and X:Bar in the same argument
list is disallowed).
Also, the righthand side, X:Int < Y:Int or X:Int == Y:Int in this case, must parse to
a unique term in the target module or theory. The only variables that may occur in the target
term are those appearing in the source term; however, they may occur multiple times or not at
all. If the source term parses to a sort S or kind [S], then the target term must parse to sort
T or kind [T] such that T and the mapping of S under the view S′belong to the same kind.
Views may also contain variable declarations. The syntax is identical to that in modules
and theories. However, its semantics is subtly different. Instead of declaring a single variable,
a declaration

<!-- manual page 162 -->
150 CHAPTER 7. MODULE OPERATIONS
var X : S.
now declares two aliases with the same name; in the lefthand side of an operator mapping, X
is an alias for X:S while in the righthand side of an operator mapping, X is an alias for X:S′
,
with S′the mapping of S under the view.
For example, we can define a view from the theory SPOSET with a strict order operation _<_
to the predefined functional module INT of integers (see Section 8.5) in such a way that the _<_
order relation of a poset is mapped to an expression using the “less than or equal” operator _<=_
on sort Int and the predefined inequality operator _=/=_ in BOOL (see Sections 8.5 and 8.1) as
follows:
view SPosetToInt from SPOSET to INT is
sort Elt to Int .
vars X Y : Elt .
op X < Y to term X <= Y and X =/= Y .
endv
Alternatively, we can specify this view without a variable declaration as
view SPosetToInt from SPOSET to INT is
sort Elt to Int .
op X:Elt < Y:Elt to term X:Int <= Y:Int and X:Int =/= Y:Int .
endv
Note that this view imposes several proof obligations to be checked by the user. In particular,
the translations by the view of the axioms in SPOSET should hold in the target. Given variables
X, Y, and Z of sort Int, the following axioms should be true in INT:
eq X <= X and X =/= X = false .
ceq X <= Z and X =/= Z = true
if X <= Y and X =/= Y /\ Y <= Z and Y =/= Z .
ceq X = Y if X <= Y and X =/= Y /\ Y <= X and Y =/= X .
Of course, since the predefined INT module indeed includes both operators _<_ and _<=_,
it is not necessary to use the feature described in the previous example. We can instead have
simpler view declarations such as the following:
view IntAsStoset from STOSET to INT is
sort Elt to Int .
endv
view IntAsToset from TOSET to INT is
sort Elt to Int .
endv
where the identity maps op _<_ to _<_ and op _<=_ to _<=_ have been omitted.
We recommend following the convention of naming views from TRIV by the name of the
sort to which Elt is mapped. Thus, a view from the theory TRIV to the module INT that sends
the sort Elt to Int should be named Int (as we shall see in Section 8.13.1, the view Int is
predefined in Maude).
view Int from TRIV to INT is
sort Elt to Int .
endv
This convention can add understandability to the specifications. As we will see in Section 7.4.4,
given a module LIST of lists parameterized by TRIV with a sort List{X}, once it is instantiated,

<!-- manual page 163 -->
7.4. PARAMETERIZED PROGRAMMING 151
e.g., with the view Int above, the sort List{X} becomes List{Int}, defining lists of integers.
Using names of views as labels in interfaces of parameterized modules (see Section 7.4.4 below)
should be avoided, since this can sometimes generate ambiguities.
We can also have views between theories, which is particularly useful to compose instan-
tiations of views to link the formal parameter of some parameterized module to some actual
parameter via some intermediate formal parameter of another parameterized module. We will
discuss the uses of these views and give some examples in the coming sections. An example of
a view whose target is a theory is the following:
view PosetToToset from POSET to TOSET is
sort Elt to Elt .
endv
As said above, identity maps can be omitted. Thus, the following definition is equivalent to
the previous one.
view PosetToToset from POSET to TOSET is
endv
In this example the PosetToTosetview represents the inclusion of the POSETtheory into TOSET.
In those cases in which a view defines a theory inclusion from TRIV into another theory, we
recommend following the convention of naming the view with the name of the target theory.
An example that will be very useful later is the inclusion of TRIV into TOSET, which is expressed
as a view as follows:
view TOSET from TRIV to TOSET is
endv
Asanexampleofviewfromanobject-orientedtheorytoanobject-orientedmodule, consider
thefollowingviewAccountfromtheobject-orientedtheoryCELL(inSection7.4.1)totheobject-
oriented module ACCOUNT (in Section 6.1.4).
view Account from CELL to ACCOUNT is
sort Elt to Int .
class Cell to Account .
attr contents to bal .
endv
Let us finish this section by commenting on some subtle issues that can arise with operator
mappings:
• Operator mappings are not applied to operators that have at least one declaration in a
module (as opposed to a theory); if a mapping applies to such an operator, an advisory is
generated. Although it does not seem to be useful, Maude does not forbid having subsort-
overloaded operators appearing in a theory and in one of its submodules. However, the
operator is considered to “belong” to the module, and therefore it cannot be mapped by
a view.
• assoc operators. Nested occurrences of associative operators may have been flattened,
or have been entered in a flattened way such as, for example, f(a, a, b, b). In order
to map this to an operator that has different attributes (perhaps including assoc) or to
a term, flattened occurrences will be temporarily unflattened into a regular term before
translation. The precise choice of unflattening is left unspecified.
• iter operators. Mapping an iter operator (see Section 4.4.2) to a non-iter operator
causes the efficient representation of towers of symbols to be expanded out, with a po-
tential exponential blow up. Mapping an iter operator to a term in which the single
argument variable occurs more than once causes a doubly exponential blow up. Maude

<!-- manual page 164 -->
152 CHAPTER 7. MODULE OPERATIONS
operates under the principle of “you asked for it, you got it” and if the expansion is too
large it will die with a virtual memory exhausted error.
• Built-in operators. The built-in operators for holding non-algebraically defined data
StringSymbol, FloatSymbol, and QuotedIdentifierSymbol have a special internal rep-
resentation for their terms, and can only be mapped to operators of identical type.
• Polymorphic operators. Polymorphic operators must map to polymorphic operators that
are polymorphic on the same arguments. Only generic mappings of the form f to f′are
considered when mapping polymorphic operators.
7.4.3 Parameterized modules
Functional modules, system modules, and object-oriented modules can be parameterized. A
parameterized functional module has syntax
fmod M{X1 :: T1 , ..., Xn :: Tn} is ... endfm
with n ≥1. Parameterized system and object-oriented modules have completely analogous
syntax.
mod M{X1 :: T1 , ..., Xn :: Tn} is ... endm
omod M{X1 :: T1 , ..., Xn :: Tn} is ... endom
The {X1 :: T1 , ..., Xn :: Tn} part is called the interface, where each pair Xi::Ti is
a parameter, and each Xi is an identifier—the parameter name or parameter label—and each
Ti is an expression that yields a theory—the parameter theory. Each parameter name in an
interface must be unique, although there is no uniqueness restriction on the parameter theories
of a module—we can have, e.g., two TRIV parameters. The parameter theories of a functional
module must be functional theories.
In a parameterized module M, all the sorts, classes, parameter constants, and statement
labels coming from theories in its interface must be qualified by their names. Thus, given
a parameter Xi::Ti, each sort S in Ti must be qualified as Xi$S, each class C in Ti must
be qualified as Xi$C, each parameter constant c in Ti must be qualified as Xi$c,
3 and each
label l of a statement occurring in Ti must be qualified as Xi$l. In fact, the parameterized
module M is flattened as follows. For each parameter Xi::Ti, a renamed copy of the theory
Ti, called Xi::Ti is included. The renaming maps each sort S to Xi$S, each class C to
Xi$C, each parameter constant c to Xi$c, and each label l of a statement occurring in Ti to
Xi$l. The renaming percolates down through nested inclusions of theories, but has no effect on
importations of modules. Thus, if Ti includes a theory T′, when the renamed theory Xi::Ti
is created and included into M, the renamed theory Xi::T′will also be created and included
into Xi::Ti.
4 However, the renaming will have no effect on modules imported by either the Ti
or T′; for example, if BOOL is imported by one of these theories, it is not renamed, but imported
in the same way into M.
For example, a parameterized module PRELIM-SET with TRIV as interface can be defined as
follows:
fmod PRELIM-SET{X :: TRIV} is
protecting BOOL .
sorts Set NeSet .
3A parameter constant is a constant that is given the attribute pconst (see Section 7.4.1). Constants without
the attribute, which we can call non-parameter constants, do not get qualified. Instead, they are imported as
such, as any other operator.
4These renamed modules are visible as names when using the show modules command (see Appendix A.12)
and will be shared, but they cannot be referred to directly in module expressions.

<!-- manual page 165 -->
7.4. PARAMETERIZED PROGRAMMING 153
subsorts X$Elt < NeSet < Set .
op empty : -> Set .
op _,_ : Set Set -> Set [assoc comm id: empty] .
op _,_ : NeSet NeSet -> NeSet [ditto] .
op _in_ : X$Elt Set -> Bool .
op _-_ : Set Set -> Set . *** set difference
var E : X$Elt .
vars S S’ : Set .
eq E, E = E .
eq E in E, S = true .
eq E in S = false [owise] .
eq (E, S) - (E, S’) = S - (E, S’) .
eq S - S’ = S [owise] .
endfm
In Maude—unlike OBJ3 and other similar languages—sorts are not systematically qualified
by their module name. This convention of not qualifying sorts may be particularly weak when
dealing with parameterized modules. However, given that Maude supports ad-hoc overload-
ing and that constants can be qualified in order to be disambiguated (see Section 3.9.3), the
problem of ambiguity in a signature is reduced to collisions of sorts. For example, in a module
one may very easily need sets of integers and sets of quoted identifiers, in which case, given
the specification of the PRELIM-SET module above, we would get two Set sorts from different
importations which would be confused into one sort. Our solution consists in qualifying pa-
rameterized sorts, not with the module expression they belong to, but with the name of the
view or views used in the instantiation of the parameterized module. Since we assume that
all views are named, these names are the ones used in the qualification. Specifically, in the
body of a parameterized module M{X1 ::T1 , ... ,Xn::Tn}, any sort S can be written in
the form S{X1 , ... ,Xn}. When the module is instantiated with views V1 ... Vn, then this
sort is instantiated to S{V1 , ... ,Vn}.
Sorts and classes declared in the parameterized module M{X1 ::T1 , ... ,Xn::Tn} may
in general be parameterized as S{Y1 , ... ,Ym} and C{Y1 , ... ,Ym}, with m≥1, and where
each Yj is an Xi. It is recommended that all sorts and classes declared in a parameterized
module be parameterized with m = n and Yj= Xj for 1 ≤j ≤n, but this is not enforced—
parameterized sorts and classes may duplicate, omit, or reorder parameters and unparameter-
ized sorts are also allowed. Although we strongly recommend it, the parameterization of sorts
and classes is optional, and therefore, for example, the above PRELIM-SET specification is a
perfectly valid parameterized module.
Thus, the previous PRELIM-SET module to define sets could instead have been specified in
a better way as follows:
fmod BASIC-SET{X :: TRIV} is
protecting BOOL .
sorts Set{X} NeSet{X} .
subsorts X$Elt < NeSet{X} < Set{X} .
op empty : -> Set{X} .
op _,_ : Set{X} Set{X} -> Set{X} [assoc comm id: empty] .
op _,_ : NeSet{X} NeSet{X} -> NeSet{X} [ditto] .
op _in_ : X$Elt Set{X} -> Bool .
op _-_ : Set{X} Set{X} -> Set{X} . *** set difference
var E : X$Elt .
vars S S’ : Set{X} .

<!-- manual page 166 -->
154 CHAPTER 7. MODULE OPERATIONS
eq E, E = E .
eq E in E, S = true .
eq E in S = false [owise] .
eq (E, S) - (E, S’) = S - (E, S’) .
eq S - S’ = S [owise] .
endfm
When this module is instantiated with the predefined view Int, the sort Set{X} becomes
Set{Int}, and when it is instantiated with the predefined view Qid (see Section 8.13.1) it
becomes Set{Qid}. In the following sections we will see additional examples of how this qual-
ification convention for the sorts of a parameterized module avoids many unintended collisions
of sort names, thus making renaming practically unnecessary.5
Constants can be parameterized just like sorts and classes, and their behavior under instan-
tiation exactly mimics that of sorts and classes. As a simple example, we consider a module
MAYBE{X :: TRIV} in which we declare a sort Maybe{X} as a supersort of the sort Elt of the
parameter theory and a constant maybe{X} of this sort Maybe{X}. This technique is useful to
declare a partial function as a total function, as we will see in the PFUN module of Section 7.4.7.
fmod MAYBE{X :: TRIV} is
sort Maybe{X} .
subsort X$Elt < Maybe{X} .
op maybe{X} : -> Maybe{X} [ctor] .
endfm
When the MAYBE module is instantiated with the predefined view Nat, the sort Maybe{X} be-
comes Maybe{Nat}, and the constant maybe{X} becomes maybe{Nat}.
The PRELIM-SET, BASIC-SET, and MAYBE modules above have only one parameter. In gen-
eral, however, parameterized modules can have several parameters. It can furthermore happen
that several parameters are declared with the same parameter theory, that is, we can have, for
example, an interface of the form {X :: TRIV, Y :: TRIV} involving two copies of the theory
TRIV. Therefore, parameters cannot be treated as normal submodules, since we do not want
them to be shared when their labels are different. We regard the relationship between the
body of a parameterized module and the interface of its parameters not as an inclusion, but as
a module constructor which is evaluated generating renamed copies of the parameters, which
are then included. For the above interface, two copies of the theory TRIV are generated, with
names X :: TRIV and Y :: TRIV. As already mentioned, in such copies of parameter theories,
sorts, classes, parameterized constants, and statement labels are renamed as follows: if Z is the
label of a parameter theory T, then each sort S in T is renamed to Z$S, each class C in T is
renamed to Z$C, each parameter constant cin T is renamed to Z$c, and each statement label
l is renamed to Z$l. All occurrences of these sorts, classes, parameter constants, and labels in
the body of the parameterized module must mention their corresponding renaming.
Let us consider as an example the following module PAIR, in which we would like to point
out the use of the qualifications for the sorts coming from each of the parameters.
fmod PAIR{X :: TRIV, Y :: TRIV} is
sort Pair{X, Y} .
op <_;_> : X$Elt Y$Elt -> Pair{X, Y} .
op 1st : Pair{X, Y} -> X$Elt .
op 2nd : Pair{X, Y} -> Y$Elt .
var A : X$Elt .
5In Section 7.4.7, we shall see how this naming convention can be easily extended to the case of parameterized
views.

<!-- manual page 167 -->
7.4. PARAMETERIZED PROGRAMMING 155
LEX-PAIR
X :: STOSET Y :: STOSET
X :: SPOSET Y :: SPOSET
X :: TAOSET Y :: TAOSET
BOOL
Figure 7.2: Structure of LEX-PAIR
var B : Y$Elt .
eq 1st(< A ; B >) = A .
eq 2nd(< A ; B >) = B .
endfm
As already mentioned, if a parameter theory is structured, this renaming process for pa-
rameter theories is carried out not only at the top level, but for the whole “theory part,” that
is, renaming subtheories but not renaming submodules. Consider, for example, the following
parameterized module defining a lexicographical ordering on pairs of elements of two totally
strictly ordered sets.
fmod LEX-PAIR{X :: STOSET, Y :: STOSET} is
sort Pair{X, Y} .
op <_;_> : X$Elt Y$Elt -> Pair{X, Y} .
op _<_ : Pair{X, Y} Pair{X, Y} -> Bool .
op 1st : Pair{X, Y} -> X$Elt .
op 2nd : Pair{X, Y} -> Y$Elt .
vars A A’ : X$Elt .
vars B B’ : Y$Elt .
eq 1st(< A ; B >) = A .
eq 2nd(< A ; B >) = B .
eq < A ; B > < < A’ ; B’ > = (A < A’) or (A == A’ and B < B’) .
endfm
Representing by boxes the modules (with initiality constraints), by ovals the theories (with
loose semantics), by triple arrows the protecting and parameter importations, and by single
arrows the including importations, we can depict the structure of the LEX-PAIR functional
module defining a lexicographic order on pairs as in Figure 7.2, where we have two copies not
only of STOSET but also of the SPOSET and TAOSET subtheories (see also Figure 7.1 in page 147),
but only one copy of the BOOL submodule.

<!-- manual page 168 -->
156 CHAPTER 7. MODULE OPERATIONS
The parameter theory of a module can be any module expression whose result is a theory.
The following module defines bags of elements with an occurrences operation that returns the
number of occurrences of a particular element in a given bag.
fmod BAG{X :: TRIV * (sort Elt to Element)} is
protecting NAT .
sorts Bag{X} NeBag{X} .
subsorts X$Element < NeBag{X} < Bag{X} .
op mt : -> Bag{X} .
op __ : Bag{X} Bag{X} -> Bag{X} [assoc comm id: mt] .
op __ : Bag{X} NeBag{X} -> NeBag{X} [ditto] .
op occurrences : X$Element Bag{X} -> Nat .
vars E E’ : X$Element .
var S : Bag{X} .
eq occurrences(E, E S) = 1 + occurrences(E, S) .
eq occurrences(E, S) = 0 [owise] .
endfm
As an example of an object-oriented parameterized module, we define a stack of elements.
We define a class Stack{X} as a linked sequence of node objects. Objects of class Stack{X}
have a single attribute first, containing the identifier of the first node in the stack. If the
stack is empty, the value of the first attribute is null. Each object of class Node{X} has an
attribute next holding the identifier of the next node—which should be null if there is no next
node—and an attribute contents to store a value of sort X$Elt. Notice that node identifiers
are of the form o(S,N), where S is the identifier of the stack object to which the node belongs,
and N is a natural number. The messages push, pop and top have as their first argument the
identifier of the object to which they are addressed, and will cause, respectively, the insertion
at the top of the stack of a new element, the removal of the top element, and the sending of a
response message elt containing the element at the top of the stack to the object making the
request.
omod OO-STACK{X :: TRIV} is
protecting INT .
protecting QID .
subsort Qid < Oid .
class Node{X} | next : Oid, contents : X$Elt .
class Stack{X} | first : Oid .
msg _push_ : Oid X$Elt -> Msg .
msg _pop : Oid -> Msg .
msg _top_ : Oid Oid -> Msg .
msg _elt_ : Oid X$Elt -> Msg .
op null : -> Oid .
op o : Oid Int -> Oid .
vars O O’ O’’ : Oid .
var E : X$Elt .
var N : Int .
rl [top] : *** top on a non-empty stack
< O : Stack{X} | first : O’ >
< O’ : Node{X} | contents : E >
(O top O’’)

<!-- manual page 169 -->
7.4. PARAMETERIZED PROGRAMMING 157
=> < O : Stack{X} | >
< O’ : Node{X} | >
(O’’ elt E) .
rl [push1] : *** push on a non-empty stack
< O : Stack{X} | first : o(O, N) >
(O push E)
=> < O : Stack{X} | first : o(O, N + 1) >
< o(O, N + 1) : Node{X} |
contents : E, next : o(O, N) > .
rl [push2] : *** push on an empty stack
< O : Stack{X} | first : null >
(O push E)
=> < O : Stack{X} | first : o(O, 0) >
< o(O, 0) : Node{X} | contents : E, next : null > .
rl [pop] : *** pop on a non-empty stack
< O : Stack{X} | first : O’ >
< O’ : Node{X} | next : O’’ >
(O pop)
=> < O : Stack{X} | first : O’’ > .
endom
Notice that top and pop messages are not received if the stack is empty.
Wemaywanttodefinestacksstoringnotjustdataelementsofaparticularsort, butactually
objects in a particular class. We can define an object-oriented module with the intended
behavior as the parameterized module OO-STACK2 below, which is parameterized by the object-
oriented theory CELL introduced in Section 7.4.1. Notice that the main difference with respect
to the previous STACK version is in the attribute node, that keeps the identifier of the object
where the contents can be found instead of the attribute contents that provided direct access
to those contents.
omod OO-STACK2{X :: CELL} is
protecting INT .
protecting QID .
subsort Qid < Oid .
class Node{X} | next : Oid, node : Oid .
class Stack{X} | first : Oid .
msg _push_ : Oid Oid -> Msg .
msg _pop : Oid -> Msg .
msg _top_ : Oid Oid -> Msg .
msg _elt_ : Oid X$Elt -> Msg .
op null : -> Oid .
op o : Oid Int -> Oid .
vars O O’ O’’ O’’’ : Oid .
var E : X$Elt .
var N : Int .
rl [top] : *** top on a non-empty stack
< O : Stack{X} | first : O’ >
< O’ : Node{X} | node : O’’ >

<!-- manual page 170 -->
158 CHAPTER 7. MODULE OPERATIONS
< O’’ : X$Cell | contents : E >
(O top O’’’)
=> < O : Stack{X} | >
< O’ : Node{X} | >
< O’’ : X$Cell | >
(O’’’ elt E) .
rl [push1] : *** push on a non-empty stack
< O : Stack{X} | first : o(O, N) >
(O push O’)
=> < O : Stack{X} | first : o(O, N + 1) >
< o(O, N + 1) : Node{X} |
next : o(O, N), node : O’ > .
rl [push2] : *** push on an empty stack
< O : Stack{X} | first : null >
(O push O’)
=> < O : Stack{X} | first : o(O, 0) >
< o(O, 0) : Node{X} | next : null, node : O’ > .
rl [pop] : *** pop on a non-empty stack
< O : Stack{X} | first : O’ >
< O’ : Node{X} | next : O’’ >
(O pop)
=> < O : Stack{X} | first : O’’ > .
endom
execution examples.
Module instantiation will be explained in the next section, and then we shall see some
7.4.4 Module instantiation
Instantiation is the process by which actual parameters are bound to the formal parameters
of a parameterized module and a new module is created as a result. This can be seen in fact
as the evaluation of a module expression. The instantiation requires a view from each formal
parameter to its corresponding actual parameter. Each such view is then used to bind the
names of sorts, operators, etc. in the formal parameters to the corresponding sorts, operators
(or expressions), etc. in the actual target.
The instantiation of a parameterized module must be made with views explicitly defined
previously. Thus, given the views Int (from TRIV to INT) and IntAsStoset (from STOSET to
INT), both introduced in Section 7.4.2, we can define sets of integers with the module expression
BASIC-SET{Int}, and lexicographically ordered pairs of integers with LEX-PAIR{IntAsStoset,
IntAsStoset}.
As mentioned in Section 7.4.2, there are also views from theories to theories. Using such
views we can, for example, instantiate the module BASIC-SET with the view TOSET (from TRIV
to TOSET) given also in Section 7.4.2. The result is a module BASIC-SET{TOSET} which is
still parameterized, but now by the theory TOSET. We can instantiate it again with a view
from TOSET to some other theory or module, for example, IntAsToset (from TOSET to INT),
obtaining the module BASIC-SET{TOSET}{IntAsToset}, which defines sets of integers. Note
that certain new operations, which would not be meaningful in the original BASIC-SET module,
could now be defined in a totally parametric way in an extension of BASIC-SET{TOSET}. For
example, we could define in this way a maximum function

<!-- manual page 171 -->
7.4. PARAMETERIZED PROGRAMMING 159
op max : NeSet{TOSET}{X} -> X$Elt .
as done in the SET-MAX module later in this section.
Another interesting use of parameterized modules is the linking of parameters. Suppose that
we wish to define lists of sets of elements. We may define a module SET-LIST parameterized
by the theory TRIV that imports the module BASIC-SET and declares the sort SetList{X}
with constructors nil and _;_. Note however that BASIC-SET is also a parameterized module,
which must be instantiated to be imported. In cases like this one, we can use the label of the
parameter to link the parameter of the module with the parameter of the submodule. Once
the module is instantiated, the parameterized submodule gets instantiated with the same view.
Thus, if the module SET-LIST below is instantiated by, say, the view Int to define lists of sets
of integers, the submodule BASIC-SET also gets instantiated with the same view, providing a
definition of sets of integers.6
fmod SET-LIST{X :: TRIV} is
protecting BASIC-SET{X} .
sort SetList{X} .
subsort Set{X} < SetList{X} .
op nil : -> SetList{X} [ctor] .
op _;_ : SetList{X} SetList{X} -> SetList{X}
[ctor assoc id: nil] .
endfm
As another example, let us consider the following modules MONOMIAL and POLYNOMIAL,
defining, respectively, monomials on a set of variables and polynomials on a commutative ring
and a set of variables. First, the module MONOMIAL defines monomials as terms of the form
X ^ N, with X a variable7 and N a nonzero natural number indicating the power to which the
variable is raised, and with an empty syntax multiplication operation__
on monomials.
fmod MONOMIAL{X :: TRIV} is
protecting NAT .
sorts Pow{X} Mon{X} .
subsorts Pow{X} < Mon{X} .
*** multiplication
op __ : Mon{X} Mon{X} -> Mon{X} [assoc comm] .
op _^_ : X$Elt NzNat -> Pow{X} .
var X : X$Elt .
vars N M : NzNat .
eq (X ^ N) (X ^ M) = X ^ (N + M) .
endfm
Once we have the specification of monomials, we can specify polynomials as monomials
with coefficients in some commutative ring, and with addition and multiplication operations.
Thus, for specifying polynomials on a ring and a set of variables in a module POLYNOMIAL, we
need to import the above module MONOMIAL. But notice that POLYNOMIAL is parameterized by
two theories: RING, for the coefficients, and TRIV, for the variables. Since we need to import
monomials on the same set of variables, we need to bind or link such parameters. This linking
is done by means of the label X of the parameter theory X :: TRIV.
6In Section 7.4.7, we shall introduce the notion of parameterized views, a more convenient way of defining
this kind of structures.
7Note that a variable in a monomial or polynomial is a constant, not a mathematical variable in the Maude
sense. That is, in this example variables are understood as names. Of course, in Maude we can also define a
variable X:X$Elt in the parameter sort to which variables belong as constants, or, more generally, variables such
as P:Poly{R, X}. In this context such mathematical variables can be distinguished from variables as names by
referring to them as metavariables.

<!-- manual page 172 -->
160 CHAPTER 7. MODULE OPERATIONS
fmod POLYNOMIAL{R :: RING, X :: TRIV} is
protecting MONOMIAL{X} .
sorts Poly{R, X} .
subsorts R$Ring < Poly{R, X} .
*** multiplication
op __ : Poly{R, X} Poly{R, X} -> Poly{R, X} [assoc comm] .
*** addition
op _++_ : Poly{R, X} Poly{R, X} -> Poly{R, X} [assoc comm] .
op --_ : Poly{R, X} -> Poly{R, X} .
op __ : R$Ring Mon{X} -> Poly{R, X} .
vars A B : R$Ring .
vars U V : Mon{X} .
vars P Q R : Poly{R, X} .
eq P ++ z = P .
eq P ++ (-- P) = z .
eq P e = P .
eq -- (P ++ Q) = (-- P) ++ (-- Q) .
eq -- (A U) = (- A) U .
eq P (Q ++ R) = (P Q) ++ (P R) .
eq z U = z .
eq z P = z .
eq A (B U) = (A B) U .
eq (A U) ++ (B U) = (A ++ B) U .
eq (A U) (B V) = (A B) (U V) .
eq A B = A * B .
eq A ++ B = A + B .
endfm
If the module POLYNOMIAL is instantiated with, say, views RingToRat and Qid, the submodule
MONOMIAL then gets automatically instantiated with Qid, thanks to the binding of the parame-
ters.
As an additional example, let us give a more concise definition of the parameterized module
LEX-PAIR{X :: STOSET, Y :: STOSET} given in Section 7.4.3 using these ideas as follows:
view STOSET from TRIV to STOSET is
endv
fmod LEX-PAIR{X :: STOSET, Y :: STOSET} is
protecting PAIR{STOSET, STOSET}{X, Y} .
op _<_ : Pair{STOSET, STOSET}{X, Y} Pair{STOSET, STOSET}{X, Y} -> Bool .
vars A A’ : X$Elt .
vars B B’ : Y$Elt .
eq < A ; B > < < A’ ; B’ > = (A < A’) or (A == A’ and B < B’) .
endfm
In Section 7.3, we presented a NAT-LIST-MAX module in which we defined a max function
that returns the greatest element of a list of natural numbers. However, we can define such
a function on lists or sets of any type of elements as long as there is a total order relation
available for them. Let us consider the following module SET-MAX, parameterized by the theory
TOSET (see Section 7.4.1). Given a non-empty finite set of elements in a totally ordered set, the
operation max returns the maximum element in the set. Note that we have used the or-else
operator for short-circuit disjunction from the EXT-BOOL module to improve the efficiency of
the calculation.

<!-- manual page 173 -->
7.4. PARAMETERIZED PROGRAMMING 161
fmod SET-MAX{T :: TOSET} is
protecting BASIC-SET{TOSET}{T} .
protecting EXT-BOOL .
op max : NeSet{TOSET}{T} -> T$Elt .
var E : T$Elt .
var S : Set{TOSET}{T} .
eq max(E, S)
= if S == empty or-else max(S) < E
then E
else max(S)
fi .
endfm
We can now calculate the maximum of a set of integers by instantiating this module with
the view IntAsToset introduced in Section 7.4.2. Notice that in this example we need an extra
set of parentheses to disambiguate between the operator max just defined and the associative
operator max on integers.
fmod INT-SET-MAX is
protecting SET-MAX{IntAsToset} .
endfm
Maude> red max((4, 3, 5, 2, 1)) .
result NzNat: 5
Similarly, wecancalculatethegreatestelementinsetsofanytypewithatotalorderrelation;
for example, sets of strings, by using the view StringAsToset also introduced in Section 7.4.2:
fmod STRING-SET-MAX is
protecting SET-MAX{StringAsToset} .
endfm
Maude> red max("four", "three", "five", "two", "one") .
result String: "two"
Notice that, if we have several parameters, we can instantiate the parameterized module or
theory with some views going to theories and others going to modules. The result in such case
is the expected one, that is, we get a module or theory parameterized by the targets of those
views going to theories. For example, the module RAT-POLY below gives us a specification of
the polynomials with rational coefficients by just importing the module POLYNOMIAL introduced
above instantiated with the view RingToRat from the theory RING to the functional module
RAT (see Section 7.4.2).
fmod RAT-POLY{X :: TRIV} is
protecting POLYNOMIAL{RingToRat, X} .
endfm
We can then define the polynomials with rational coefficients and with quoted identifiers as
variablesbyinstantiatingthemoduleRAT-POLYwiththefollowing Qidview, whichispredefined
in Maude (see Section 8.13.1).
view Qid from TRIV to QID is
sort Elt to Qid .
endv

<!-- manual page 174 -->
162 CHAPTER 7. MODULE OPERATIONS
fmod QID-RAT-POLY is
pr RAT-POLY{Qid} .
endfm
Let us reduce as an example the following polynomial expression:
Maude> red in QID-RAT-POLY :
(((2 / 3) ((’X ^ 2) (’Y ^ 3)))
++ ((7 / 5) ((’Y ^ 2) (’Z ^ 5))))
(((1 / 7) (’U ^ 2))
++ (1 / 2)) .
result Poly{RingToRat, Qid}:
(1/3 (’X ^ 2) ’Y ^ 3)
++ (1/5 (’U ^ 2) (’Y ^ 2) ’Z ^ 5)
++ (2/21 (’U ^ 2) (’X ^ 2) ’Y ^ 3)
++ (7/10 (’Y ^ 2) ’Z ^ 5)
Summarizing, aparameterizedmoduleM{X1 ::T1 , ... ,Xn::Tn}withnfreeparameters
is instantiated by the module expression M{A1, ..., An}, where each Ai is an instance of one
of the following three alternatives:
• The name Yj of a parameter of the correct theory from the module enclosing the module
expression. In this case the parameter becomes a bound parameter in the module resulting
from the instantiation. Each sort Xi$S is mapped to Yj$S, each class Xi$C is mapped
to Yj$C, each parameter constant Xi$c is mapped to Yj$c, and each Xi occurring as a
parameter in a parameterized sort becomes Yj (and similarly for statement labels).
• The name of a view V with a theory as target with the correct source theory. In this case,
the parameter becomes a free parameter with V’s target theory in the module resulting
from the instantiation.
• The name of a view V with a module as target with the correct source theory. In this
case, the parameter disappears. Each sort, class or parameter constant Xi$A is mapped
to A′, where A′ is the mapping of A under V. Each Xi occurring as a parameter in a
parameterized sort, class or constant becomes V. Each statement label Xi$l is mapped
to l′, where l′is the mapping of l under the view V.
Parameterized modules with free parameters cannot be imported: first all of the free pa-
rameters must be instantiated away. Parameterized modules with bound parameters may only
be imported, since they were created for module expressions in a context where the parameters
are bound by an enclosing parameterized module.
Parameterizedfunctionalmodulesmaybeinstantiatedwithviewsthathavesystemmodules
as their targets; then the instantiated module is promoted to a system module.
Parameterized modules cannot be summed, even if all the parameters are bound. Parame-
terized modules may be renamed, but the renaming must not affect any sorts, classes, messages,
or operators coming from a parameter theory. The result of renaming a parameterized module
is a parameterized module with the same parameters, and we can use it as any other param-
eterized module; for example, we can instantiate it with a view, or bind its parameters to the
parameters of the module in which the module expression is being imported, as in the following
example, where we rename the SET-LIST parameterized module above.
fmod MY-SET-LIST{Y :: TRIV} is
pr (SET-LIST
* (sort Set{X} to MySet{X},
op __ : SetList{X} SetList{X} -> SetList{X} to _._)) {Y} .
endfm

<!-- manual page 175 -->
7.4. PARAMETERIZED PROGRAMMING 163
fmod MY-QID-SET-LIST is
protecting MY-SET-LIST{Qid} .
endfm
TheSET-LISTmodulehasonlyfreeparametersandsoitcanberenamed; however, itsrenaming
imports the renaming of BASIC-SET{X} which has a bound parameter. Note that the parameter
of the sorts appearing in the renaming of the SET-LIST module is X, since this is the label of
the parameter in such module. We have used label Y for the parameter of MY-SET-LIST to
emphasize this fact, although they could be the same.
Allowing renaming of modules with bound parameters requires that renamings be capable of
instantiation; thatis, parameterizedsortorclassnamesinsidearenaminghavetheirparameters
instantiated, with an extra pair of curly brackets being added in the case of instantiation by a
view with a theory as target.
Let us illustrate these ideas. When, due to instantiation by a view with a theory as tar-
get, a bound parameter in a renamed module escapes and needs to be rebound by an extra
instantiation, the extra instantiation is inserted before rather than after the renaming. Let us
consider the following example, where we use the views TOSET, from the theory TRIV to the
theory TOSET, and IntAsToset, from the theory TOSET to the predefined module INT, both
described in Section 7.4.2.
fmod RENAMING-PAR-MOD-A{X :: TRIV} is
sort Foo{X} .
op f : Foo{X} -> Foo{X} .
endfm
fmod RENAMING-PAR-MOD-B{X :: TRIV} is
extending RENAMING-PAR-MOD-A{X} .
sort Bar{X} .
op g : Bar{X} -> Foo{X} .
endfm
fmod RENAMING-PAR-MOD-C is
pr (RENAMING-PAR-MOD-B
* (sort Foo{X} to Foo’{X},
sort Bar{X} to Bar’{X},
op f : Foo{X} -> Foo{X} to f’,
op g : Bar{X} -> Foo{X} to g’)) {TOSET} {IntAsToset} .
endfm
In this case, the module RENAMING-PAR-MOD-A gets instantiated before it is renamed:
RENAMING-PAR-MOD-A{TOSET}{IntAsToset}
* (sort Foo{TOSET}{IntAsToset} to Foo’{TOSET}{IntAsToset},
op f : [Foo{TOSET}{IntAsToset}] -> [Foo{TOSET}{IntAsToset}] to f’)
Passing parameters from an enclosing module in nonfinal instantiations is prohibited. This
restriction avoids many subtle issues. Thus:
fmod ILLEGAL-INST{X :: RING, Y :: POSET} is
protecting POLYNOMIAL{X, POSET}{Y} .
endfm
is illegal, because X occurs in the nonfinal instantiation POLYNOMIAL{X, POSET}. With appro-
priate views, this example can be correctly written as follows:
view RING from RING to RING is
endv

<!-- manual page 176 -->
164 CHAPTER 7. MODULE OPERATIONS
view POSET from TRIV to POSET is
endv
fmod LEGAL-INST{X :: RING, Y :: POSET} is
protecting POLYNOMIAL{RING, POSET}{X, Y} .
endfm
Another way of viewing this restriction is that parameters from an enclosing module and
views with theories as targets may not occur in the same instantiation. Note that views with
theories as targets may never occur in a final instantiation (otherwise there would be free
parameters in an import) and must occur in any nonfinal instantiation (otherwise there would
be no free parameters for the next instantiation).
We conclude this section with an example in which a parameterized object-oriented module
parameterized with an object-oriented theory, is instantiated. By instantiating the object-
oriented module OO-STACK2 given in Section 7.4.3, we can obtain a specification of a stack
of banking accounts. Given the Account view from the object-oriented theory CELL (in Sec-
tion 7.4.1) to the object-oriented module ACCOUNT (in Section 6.1.4) defined in Section 7.4.2 we
could directly import the module expression OO-STACK2{Account}. We make the example a
bit more interesting by adding some renamings to some of the features of the specification.
omod ACCOUNT-OO-STACK2 is
inc OO-STACK2{Account}
* (attr first to head,
attr bal to balance,
msg _elt_ to element,
sort Int to Integer) .
endom
Now we can do the following rewriting on the module resulting from the instantiation.
Maude> rew < ’stack : Stack{Account} | head : null >
< ’A-73728 : Account | balance : 5000 >
< ’A-06238 : Account | balance : 2000 >
< ’A-28381 : Account | balance : 15000 >
(’stack push ’A-73728)
(’stack push ’A-06238)
(’stack push ’A-28381)
(’stack top ’A-06238)
(’stack pop) .
result Configuration :
element(’A-28381,15000)
< ’A-06238 : Account | balance : 2000 >
< ’A-28381 : Account | balance : 15000 >
< ’A-73728 : Account | balance : 5000 >
< ’stack : Stack{Account}| head : o(’stack, 1)>
< o(’stack, 0) : Node{Account} | next : null, node : ’A-06238 >
< o(’stack, 1) : Node{Account} | next : o(’stack, 0), node : ’A-73728 >
7.4.5 Lists
There are different ways of building lists. One possibility is to begin with the empty list
and the singleton lists, and then use the concatenation operation to get bigger lists. However,
concatenation cannot be a free list constructor, because it satisfies an associativity equation and

<!-- manual page 177 -->
7.4. PARAMETERIZED PROGRAMMING 165
has the empty list as identity. This approach will be used in the predefined module for generic
lists described in Section 8.14.1, and appears in many similar examples throughout this book.
Given the support for equational attributes (associativity, commutativity, etc.) in Maude, as
explained in Section 4.4.1, one can argue that this is indeed the most natural specification for
lists in Maude.
Here we use instead the two standard free constructors for lists that can be found in many
functional programming languages: the empty list nil, here denoted [], and the cons operation
that adds an element to the beginning of a list, here denoted with the mixfix syntax _:_. This
approach facilitates proving properties of lists by structural induction in Maude’s inductive the-
orem prover (ITP), and provides a simple basis for specifying sorted lists and sorting operations
on them in Section 7.4.6.
As usual, head and tail are the selectors associated with the _:_ constructor. Since they
are not defined on the empty list, we avoid their partiality by means of a subsort NeList of
non-empty lists.
fmod LIST-CONS{X :: TRIV} is
protecting NAT .
sorts NeList{X} List{X} .
subsort NeList{X} < List{X} .
op [] : -> List{X} [ctor] .
op _:_ : X$Elt List{X} -> NeList{X} [ctor] .
op tail : NeList{X} -> List{X} .
op head : NeList{X} -> X$Elt .
var E : X$Elt .
var N : Nat .
vars L L’ : List{X} .
eq tail(E : L) = L .
eq head(E : L) = E .
Three interesting operations on lists are list concatenation (here denoted with mixfix syntax
_++_), thelengthofalist, andreversingalist. ThelengthoperatorhasaresultofsortNat, that
comes from the predefined module NAT, imported in protecting mode. These three operations
are defined as usual by structural induction on the two constructors, with an equation for the
empty base case and another for the cons case E : L.
Here we are not concerned with efficiency and therefore we just specify the operations in
a simple way, without using, for example, tail-recursive auxiliary operations in the style of
Section 8.14.1.
op _++_ : List{X} List{X} -> List{X} .
op length : List{X} -> Nat .
op reverse : List{X} -> List{X} .
eq [] ++ L = L .
eq (E : L) ++ L’ = E : (L ++ L’) .
eq length([]) = 0 .
eq length(E : L) = 1 + length(L) .
eq reverse([]) = [] .
eq reverse(E : L) = reverse(L) ++ (E : []) .

<!-- manual page 178 -->
166 CHAPTER 7. MODULE OPERATIONS
In this specification of generic lists we also add two operations that will be useful later, in
Section 7.4.6, when sorting lists: take_from_ and throw_from_. The first one builds a list by
taking the first nelements of the given list, while the second one deletes the first nelements of
the given list. Both of them are defined by structural induction on both arguments, the base
case being when either the first is 0 or the second is empty. As usual, s_ denotes the successor
operator on natural numbers.
op take_from_ : Nat List{X} -> List{X} .
op throw_from_ : Nat List{X} -> List{X} .
eq take 0 from L = [] .
eq take N from [] = [] .
eq take s(N) from (E : L) = E : take N from L .
eq throw 0 from L = L .
eq throw N from [] = [] .
eq throw s(N) from (E : L) = throw N from L .
endfm
The following sample reduction shows the result of reversing a list of character strings.
fmod LIST-CONS-TEST is
protecting LIST-CONS{String} .
endfm
Maude> red reverse("one" : "two" : "three" : []) .
result NeList{String}: "three" : "two" : "one" : []
7.4.6 Sorted lists
In order-sorted equational specifications, subsorts must be defined by means of constructors,
but it is not possible to have a subsort of sorted lists, for example, defined by a property
over lists; a more expressive formalism is needed. Membership equational logic allows subsort
definition by means of conditions involving equations and/or sort predicates. In this example
we use this technique to define a subsort of sorted lists, included in the sort of lists imported
from the module LIST-CONS in Section 7.4.5. Furthermore, we will also specify here different
well-known sorting algorithms.
Parameterized sorted lists need a stronger requirement than TRIV, because a total order
over the elements to be sorted is needed. Since repetitions pose no problems for sorting a list,
the order relation should be non-strict, like in the NSTOSET theory introduced in Section 7.4.1.
However, for the specification of the sorting algorithms, it is more convenient to use also the
strict version of the order. For these reasons, we will use as requirement for parameterized
sorted lists the theory TOSET, also introduced in Section 7.4.1.
The parameterized module for sorted lists imports the parameterized list module. However,
note that we want lists over a totally ordered set, instead of lists over any set; therefore, first
we partially instantiate LIST-CONS with an inclusion view from the theory TRIV to the theory
TOSET.
view TOSET from TRIV to TOSET is
endv
We are still left with a parameterized module and corresponding dependent sorts, but now with
respecttotheTOSETrequirement. ThisisthereasonjustifyingthenotationLIST-CONS{TOSET}{X}

<!-- manual page 179 -->
7.4. PARAMETERIZED PROGRAMMING 167
in the protecting importation below, as well as NeList{TOSET}{X} and List{TOSET}{X} as
names of the imported sorts.
Notice the three (conditional) membership axioms defining the subsort SortedList{X}: the
empty and singleton lists are always sorted, and a longer list is sorted when the first element
is less than or equal to the second, and the list without the first element is also sorted.
fmod SORTED-LIST{X :: TOSET} is
protecting LIST-CONS{TOSET}{X} .
sorts SortedList{X} NeSortedList{X} .
subsorts NeSortedList{X} < SortedList{X} < List{TOSET}{X} .
subsort NeSortedList{X} < NeList{TOSET}{X} .
vars N M : X$Elt .
vars L L’ : List{TOSET}{X} .
vars OL OL’ : SortedList{X} .
var NEOL : NeSortedList{X} .
mb [] : SortedList{X} .
mb (N : []) : NeSortedList{X} .
cmb (N : NEOL) : NeSortedList{X} if N <= head(NEOL) .
Aspartofthismodule,wealsodefineseveralwell-knownsortingoperations: insertion-sort,
quicksort, and mergesort, based on appropriate auxiliary operations. The important point
is that we are able to give finer typing to all these sorting operations than the usual typing
in other algebraic specification frameworks or functional programming languages. For exam-
ple, insertion-sort is declared as an operation from List{TOSET}{X} to SortedList{X},
instead of the much less informative typing from List{TOSET}{X} to List{TOSET}{X}. The
same applies to each of the auxiliary operations. Furthermore, a function that requires its input
argument to be a sorted list can now be defined as a total function, whereas in less expressive
typing formalisms it would have to be either partial, or to be defined with exceptional behavior
on the erroneous arguments.
The operation insert-list inserts an element in the appropriate position of an already
sorted list, so that the resulting list is also sorted. The sorting operation insertion-sort
recursively sorts the list without the first element and then calls insert-list, which inserts
the missing element in the correct position.
op insertion-sort : List{TOSET}{X} -> SortedList{X} .
op insert-list : SortedList{X} X$Elt -> SortedList{X} .
eq insertion-sort([]) = [] .
eq insertion-sort(N : L) = insert-list(insertion-sort(L), N) .
eq insert-list([], M) = M : [] .
ceq insert-list(N : OL, M) = M : N : OL if M <= N .
ceq insert-list(N : OL, M) = N : insert-list(OL, M) if N < M .
The sorting operation mergesort splits a given list in half by means of the operations
take_from_ and throw_from_ described in Section 7.4.5 above, recursively sorts each sublist,
and then calls the commutative merge operation on the sorted sublists to obtain the final
sorted result. In Section 8.14.6 on sortable lists there is a more efficient (albeit more complex)
definition of the mergesort algorithm on lists.
op mergesort : List{TOSET}{X} -> SortedList{X} .

<!-- manual page 180 -->
168 CHAPTER 7. MODULE OPERATIONS
op merge : SortedList{X} SortedList{X} -> SortedList{X} [comm] .
eq mergesort([]) = [] .
eq mergesort(N : []) = N : [] .
ceq mergesort(L)
= merge(mergesort(take (length(L) quo 2) from L),
mergesort(throw (length(L) quo 2) from L))
if length(L) > 1 .
eq merge(OL, []) = OL .
ceq merge(N : OL, M : OL’) = N : merge(OL, M : OL’) if N <= M .
Finally, quicksort works on a list by separating its elements into those smaller than the
first element (taken as the pivot) and those bigger than the first, recursively sorts each of the
resulting lists, and simply puts them together by concatenating them with the pivot in the
middle.
op quicksort : List{TOSET}{X} -> SortedList{X} .
op leq-elems : List{TOSET}{X} X$Elt -> List{TOSET}{X} .
op gr-elems : List{TOSET}{X} X$Elt -> List{TOSET}{X} .
eq quicksort([]) = [] .
eq quicksort(N : L)
= quicksort(leq-elems(L,N)) ++ (N : quicksort(gr-elems(L,N))) .
eq leq-elems([], M) = [] .
ceq leq-elems(N : L, M) = N : leq-elems(L, M) if N <= M .
ceq leq-elems(N : L, M) = leq-elems(L, M) if M < N .
eq gr-elems([], M) = [] .
ceq gr-elems(N : L, M) = gr-elems(L, M) if N <= M .
ceq gr-elems(N : L, M) = N : gr-elems(L, M) if M < N .
endfm
We now apply the sorting operations to lists of natural numbers.
view NatAsToset from TOSET to NAT is
sort Elt to Nat .
endv
fmod SORTED-LIST-TEST is
protecting SORTED-LIST{NatAsToset} .
endfm
Maude> red insertion-sort(5 : 4 : 3 : 2 : 1 : 0 : []) .
result NeSortedList{NatAsToset}: 0 : 1 : 2 : 3 : 4 : 5 : []
Maude> red mergesort(5 : 3 : 1 : 0 : 2 : 4 : []) .
result NeSortedList{NatAsToset}: 0 : 1 : 2 : 3 : 4 : 5 : []
Maude> red quicksort(0 : 1 : 2 : 5 : 4 : 3 : []) .
result NeSortedList{NatAsToset}: 0 : 1 : 2 : 3 : 4 : 5 : []
7.4.7 Parameterized views
Suppose we have defined modules LIST{X :: TRIV} and SET{X :: TRIV}, specifying, respec-
tively, lists and sets, and suppose that we need, e.g., the data type of lists of sets of natural

<!-- manual page 181 -->
7.4. PARAMETERIZED PROGRAMMING 169
numbers. Typically, we would first instantiate the module SET with a view, say Nat, from TRIV
to the module NAT mapping the sort Elt to the sort Nat, thus getting the module SET{Nat} of
sets of natural numbers. Then, we would instantiate the module specifying lists with a view,
say NatSet, from TRIV to SET{Nat}, obtaining the module LIST{NatSet}. But, what if we
need now the data type of lists of sets of Booleans? Should we repeat the whole process again?
One possibility is to define a combined module SET-LIST{X :: TRIV}. But what if we later
want stacks of sets instead of lists of sets?
We can greatly improve the reusability of specifications by using parameterized views. Let
us consider the following parameterized view Set from TRIV to SET, which maps the sort Elt
to the sort Set{X}.
view Set{X :: TRIV} from TRIV to SET{X} is
sort Elt to Set{X} .
endv
With this kind of views we can keep the parameter part of the target module still as a
parameter. We can now have lists of sets, stacks of sets, and so on, for any instance of TRIV, by
instantiating the appropriate parameterized module with the appropriate view. For example,
given the view Nat above, we can have the module LIST{Set{Nat}} of lists of sets of natural
numbers, or lists of sets of Booleans with LIST{Set{Bool}}, given a view Bool from TRIV to
the predefined module BOOL. Similarly, we can have STACK{Set{Nat}} or STACK{Set{Bool}}.
We can also link the parameter of a module like LIST{Set{X}} to the parameter of the
module in which it is being included. That is, we can, for example, declare a module of the
form
fmod GENERIC-SET-LIST{X :: TRIV} is
protecting LIST{Set{X}} .
endfm
Then, instantiating the parameterized module GENERIC-SET-LIST with a view V from TRIV
to another module or theory results in a module with name GENERIC-SET-LIST{V}, which
includes the module LIST{Set{V}}. Note that even with parameterized views we still follow
conventions for module interfaces and for sort names (see Section 7.4). The only difference is
that now, instead of having simple view names, we must consider names of views which are
parameterized.
The use of parameterized views in the instantiation of parameterized modules allows very
reusable specifications. For example, a very simple way of specifying (finite) partial functions
is to see a partial function as a set of input-result pairs.8 Of course, for such a set to represent
a function there cannot be two pairs associating different results with the same input value.
We show later in this section (in the module PFUN below) how this property can be specified
by means of appropriate membership axioms. Note, however, that since membership axioms
cannot be given on associative operators over sorts (see Section 21.3.8), we cannot use either
the specification of sets described in Section 7.4.3 or the predefined module in Section 8.14.2.
Let us consider instead the following module:
fmod SET-KIND{X :: TRIV} is
sorts NeKSet{X} KSet{X} .
subsort X$Elt < NeKSet{X} < KSet{X} .
op empty : -> KSet{X} [ctor] .
op _,_ : KSet{X} KSet{X} ~> KSet{X} [ctor assoc comm id: empty] .
mb NS:NeKSet{X}, NS’:NeKSet{X} : NeKSet{X} .
eq E:X$Elt, E:X$Elt = E:X$Elt . *** idempotency
8An alternative specification is available in Maude’s prelude, see Section 8.15.

<!-- manual page 182 -->
170 CHAPTER 7. MODULE OPERATIONS
endfm
Here the operator_,_ is declared at the kind level—notice the different form of the arrow in its
declaration—together with a membership axiom, that is logically equivalent to the declaration
op _,_ : NeKSet{X} NeKSet{X} -> NeKSet{X} .
at the sort level.
WecanthenspecifysetsofpairsbyinstantiatingthisSET-KINDmodulewithaparameterized
view from TRIV to the parameterized module PAIR{X, Y} defining pairs of elements introduced
in Section 7.4.3. The appropriate parameterized view can be defined as follows:
view Pair{X :: TRIV, Y :: TRIV} from TRIV to PAIR{X, Y} is
sort Elt to Pair{X, Y} .
endv
A partial function can be lifted to a total function by adding a special value to its codomain,
to be used as the result for the input elements for which the function is not defined. For this
we make good use of the parameterized module MAYBE, introduced in Section 7.4.3, which adds
a supersort and a new element maybe{X} to this supersort; in this application, the constant
maybe{X} is renamed to undefined.
We are now ready to give the specification of partial functions. The sets representing
the domain and codomain of the function are given by TRIV parameters, and then the set
of tuples is provided by the imported module expression SET-KIND{Pair{X, Y}} with sorts
KSet{Pair{X, Y}} and NeKSet{Pair{X, Y}}. We define operations dom and im returning,
respectively, the domain and image of a set of pairs. The dom operation will be used for
checking whether there is already a pair in a set of pairs with a given input value. With these
declarations we can define the sort PFun{X, Y} as a subsort of KSet{Pair{X, Y}}, by adding
the appropriate membership axioms specifying those sets that satisfy the required property.
Finally, we define operators _[_] and _[_->_] to evaluate a function at a particular element,
and to add or redefine an input-result pair, respectively. We use the Maude predefined module
SET (see Section 8.14.2) for representing the sets of elements in the domain and image of a
partial function.
fmod PFUN{X :: TRIV, Y :: TRIV} is
pr SET-KIND{Pair{X, Y}} .
pr SET{X} + SET{Y} .
pr MAYBE{Y} * (op maybe{Y} to undefined) .
sort PFun{X, Y} .
subsorts Pair{X, Y} < PFun{X, Y} < KSet{Pair{X, Y}} .
vars A D : X$Elt .
vars B C : Y$Elt .
var F : PFun{X, Y} .
var S : KSet{Pair{X, Y}} .
op dom : KSet{Pair{X, Y}} -> Set{X} . eq dom(empty) = empty .
eq dom(< A ; B >, S) = A, dom(S) .
op im : KSet{Pair{X, Y}} -> Set{Y} . eq im(empty) = empty .
eq im(< A ; B >, S) = B, im(S) .
*** domain
*** image
op empty : -> PFun{X, Y} [ctor] .

<!-- manual page 183 -->
7.4. PARAMETERIZED PROGRAMMING 171
cmb < A ; B >, < D ; C >, F : PFun{X, Y}
if < D ; C >, F : PFun{X, Y} /\ not(A in dom(< D ; C >, F)) .
op _[_] : PFun{X, Y} X$Elt -> Maybe{Y} .
op _[_->_] : PFun{X, Y} X$Elt Y$Elt -> PFun{X, Y} .
ceq (< A ; B >, F)[ A ] = B
if (< A ; B >, F) : PFun{X, Y} .
eq F [ A ] = undefined [owise] .
ceq (< A ; B >, F)[ A -> C ] = < A ; C >, F
if (< A ; B >, F) : PFun{X, Y} .
eq F [ A -> C ] = < A ; C >, F [owise] .
endfm
Now, we can instantiate the PFUN module with, for example, views Set{String} and Nat,
in order to get the finite partial functions from string sets to natural numbers by means of the
module expression PFUN{Set{String}, Nat}.
fmod SIZES is
pr PFUN{List{String}, Nat} .
endfm
Notice that views can be instantiated directly with another view rather than a parameter
from an enclosing module or view. We can then do some rewrites:
Maude> red in SIZES :
result NzNat: 2
(< "o" ; 1 >, < "o" "t" ; 2 >, < "o" "t" "t" ; 3 >) ["o" "t"] .
View nesting may be useful in some occasions. For instance, suppose that we want to
generalize our SIZES module so that lists can be of any type and we want to allow default
values. The MAYBE module in Section 7.4.3 is handy for this.
view Maybe{X :: TRIV} from TRIV to MAYBE{X} is
sort Elt to Maybe{X} .
endv
To allow lists of any type, including null elements, we can specify the following module.
fmod NULL-SIZES{X :: TRIV} is
pr PFUN{List{Maybe{X}}, Nat} * (op maybe{X} to null) .
endfm
fmod STRING-NULL-SIZES is
pr NULL-SIZES{String} .
endfm
Maude> red in STRING-NULL-SIZES :
result Maybe{Nat}: undefined
(< "o" null ; 1 >, < null "o" "t" ; 2 >, < "o" null "t" "t" ; 3 >) ["o" "t"] .
Maude> red in STRING-NULL-SIZES :
(< "o" null ; 1 >, < null "o" "t" ; 2 >, < "o" null "t" "t" ; 3 >) [null "o" "t"] .
result NzNat: 2
Parameterized views can also be instantiated on theory-views to change the theory of pa-
rameters. Assume we wanted to have our sizes structure on lists of elements with a total
order. We could use predefined theory-views STRICT-WEAK-ORDER and STRICT-TOTAL-ORDER
(see Section 8.13.3) as follows:

<!-- manual page 184 -->
172 CHAPTER 7. MODULE OPERATIONS
fmod SORTED-SIZES{X :: STRICT-TOTAL-ORDER, Y :: TRIV} is
pr PFUN{List{STRICT-WEAK-ORDER}{STRICT-TOTAL-ORDER}{X}, Y} .
endfm
And then use predefined views Nat< and String to instantiate it.
fmod NAT-SORTED-SIZES is
pr SORTED-SIZES{Nat<, String} .
endfm
Maude> red in NAT-SORTED-SIZES :
(< 0 ; "zero" >, < 1 ; "one" >, < 2 ; "two" >) [1] .
result String: "one"
Just as with module instantiation, a view instantiation that includes a theory-view is non-
final since the parameter taking the theory-view must be recaptured or instantiated in a final
instantiation. Note that as with module instantiations, bound parameters may not be passed in
a nonfinal instantiation. Furthermore for both module instantiations and view instantiations,
nested view instantiations with bound parameters may not be passed in a nonfinal instantia-
tion. In other words, view instantiations with bound parameters must obey the same rule as
bare bound parameters.
One subtle point is that an operator-to-term mapping in a parameterized view may use any
operators in the (parameterized) to-module in its to-term, including operators from parameter-
theories. Thus by extension, operator-to-operator mappings may map operators in the from-
theory to operators in a parameter-theory and sort-to-sort mappings may map sorts in a from-
theory to sorts in a parameter-theory:
fmod PAR-TH-EXAMPLE{X :: TRIV} is
sort Foo{X} .
op f : X$Elt -> Foo{X} .
endfm
view PAR-TH-EXAMPLE{X :: TRIV} from TRIV to PAR-TH-EXAMPLE{X} is
sort Elt to X$Elt . *** mapping from-theory sort to parameter-theory sort
endv
Havingparameterizedviewsallowscomplexnestingofinstantiationswithboundparameters
which can be difficult or impossible to evaluate correctly when the bound parameters are
instantiated. Thus, two technical restrictions are enforced on the use of parameterized views:
• Every parameter of a parameterized view must appear as a bound parameter in its to-
module. If a parameter does not appear in the to-module then it is effectively useless and
should be removed.
• We introduce the notion of a conflict between parameters: Two bound parameters, X
and Y, are in conflict if X is an argument to a module or view final-instantiation, and
Y is a bound parameter in some view expression that is also an argument in the same
final-instantiation.
For example, the module expressions M2{X, V{Y}} and M{V2{X, V{Y}}} both induce a
conflict on bound parameters X and Y. Conflicts on bound parameters in a module expres-
sion are propagated to the free parameters of a view that uses that module expression as
itsto-moduleandtothefreeparametersofamodulethatimportsthatmoduleexpression.
So, for example:
view V3{X :: TRIV, Y :: TRIV} from TRIV to M{X, V{Y}} is
...
endv

<!-- manual page 185 -->
7.5. EXAMPLE: DATA AGENTS 173
receives a conflict between its X and Y parameters. Conflicts are also propagated when
parameters are directly instantiated by parameters from an enclosing module or view. So,
for example,
fmod M3{A :: TRIV, B :: TRIV} is
inc M{V3{A, B}} .
...
endfm
receives a conflict between its A and B parameters because they are used to instantiate
conflicting parameters in V3.
A parameter may have a conflict with itself. For example,
fmod MMAP{X :: TRIV} is
inc MAP{X, Set{X}} .
endfm
For the most part, conflicts are quietly generated and propagated through arbitrarily deep
nesting of importations and instantiations without users being aware of them. The only
place they become important is in nonfinal instantiations, when the following restriction
is enforced: The nonfinal instantiation of a module or view may not pass theory-view ar-
guments to two parameters that have a conflict (or to a parameter that has a conflict with
itself). Passing a theory-view to one of a pair of conflicting parameters is allowed as long
as the other receives a (unparameterized) module-view (recall bound parameters are not
allowed in nonfinal instantiations). This restriction avoids situations where the recapture
of a bound parameter following instantiation by a theory-view results in the bound pa-
rameter appearing in a nonfinal instantiation. For example, if X and Y were instantiated
by theory-views TV1 and TV2, respectively, then the expression M{V2{X, V{Y}}} would
become M{V2{TV1, V{TV2}{Y}}{X}}, and Y appears as a bound parameter in a nonfinal
instantiation of view V2: V2{TV1, V{TV2}{Y}}.
7.5 Example: data agents
In this section we give an example of a simple distributed dataset that illustrates the use of
different module operations, including importations in different modes, renamings, and instan-
tiation of parameterized modules. In the example, each agent in a collection of data agents
manages a local version of a global data dictionary that maps keys to values. An agent may
only have part of the data locally, and must contact other agents to get the value of a key that
is not in its local version. To simplify the presentation, we assume that data agents work in
pairs.
This example illustrates one way of representing request-reply style of object-based pro-
gramming in Maude (see Section 6.4), and also a way of representing information about the
state of the task an object is working on when it needs to make one or more requests to other
objects in order to answer a request itself. As in the ticker example (Section 6.4.2), we define a
uniform syntax for messages. Here, messages have both a receiver and a sender in addition to
a message body, and are constructed with the msg constructor. The technique for maintaining
task information is to define a sort Request and a requests attribute that holds the set of
pending requests. The constant empty indicates that an object has no pending request. The re-
quest w4(O:Oid, C:Oid, MB:MsgBody) indicates that the object is processing a message from
C:Oid with body MB:MsgBody and is waiting for a message from O:Oid.
First, the module DATA-AGENTS-CONF extends CONFIGURATION with the uniform message
syntax and the specification of the sort Request.

<!-- manual page 186 -->
174 CHAPTER 7. MODULE OPERATIONS
mod DATA-AGENTS-CONF is
ex CONFIGURATION .
*** my msg syntax
sort MsgBody .
op msg : Oid Oid MsgBody -> Msg [ctor message] .
*** agents may be pending on requests
sort Request .
op w4 : Oid Oid MsgBody -> Request [ctor] .
endm
A data agent stores a dictionary, mapping keys to data elements. To specify such dictio-
naries, we use the predefined parameterized module MAP (see Section 8.15), renaming the main
sort as well as the lookup and update operators as follows:
MAP{K, V} * (sort Map{K, V} to Dict{K, V},
op _[_] to lookup,
op insert to update) .
Remember that the constant undefined is the result returned by the lookup operators when
the map is not defined on the given key.
We split the specification of data agents into two modules: the parameterized functional
module DATA-AGENTS-INTERFACE, which defines the interface, and the parameterized system
module DATA-AGENTS , which gives the rules for agent behavior. This illustrates a technique
for modularizing object-based system specifications in order to allow the same interface to be
shared by more than one “implementation” (rule set). We already applied this technique in
the specification of a vending machine as a system module in Section 5.1. Notice also that
DATA-AGENTS-CONF is imported in extending mode, because we add data to the old sorts, but
without making further identifications (the interface module has no equation).
mod DATA-AGENTS-INTERFACE{K :: TRIV, V :: TRIV} is
ex DATA-AGENTS-CONF .
*** messages
op getReq : K$Elt -> MsgBody [ctor] .
op getReply : K$Elt [V$Elt] -> MsgBody [ctor] .
op setReq : K$Elt V$Elt -> MsgBody [ctor] .
op setReply : K$Elt [V$Elt] -> MsgBody [ctor] .
op tellReq : K$Elt V$Elt -> MsgBody [ctor] .
op tellReply : K$Elt V$Elt -> MsgBody [ctor] .
op lookupReq : K$Elt -> MsgBody [ctor] .
op lookupReply : K$Elt [V$Elt] -> MsgBody [ctor] .
endm
In a request-reply style of interaction, message body constructors come in pairs. For exam-
ple, (lookupReq, lookupReply) and (tellReq, tellReply) are the message body pairs used
when a customer interacts with a data agent in order to access and set data values. Similarly,
(getReq, getReply) and (setReq, setReply) constitute the message body pairs for an agent
to access and set data values from a pal.
A data agent has class identifier DataAgent. In addition to the requests attribute, each
data agent has a data attribute holding the agent’s local version of the data dictionary, and a
pal attribute holding the identifier of the other agent. If sam and joe are collaborating data
agents, then their initial state might look like
< sam : DataAgent | data : empty, pal : joe, requests : empty >
< joe : DataAgent | data : empty, pal : sam, requests : empty >

<!-- manual page 187 -->
7.5. EXAMPLE: DATA AGENTS 175
The module DATA-AGENTS specifies a data agent’s behavior by giving a rule for handling
each type of message it expects to receive (other messages will simply be ignored).
SinceweareaddingrulesactingonthesortConfiguration,comingfromtheCONFIGURATION
module via DATA-AGENTS-CONF, we need to make explicit that such modules are imported in
including mode. We also import in protecting mode the predefined parameterized module
SET, instantiated with the following Request view, to define the sets of requests stored in the
requests attribute.
view Request from TRIV to DATA-AGENTS-CONF is
sort Elt to Request .
endv
mod DATA-AGENTS{K :: TRIV, V :: TRIV} is
inc DATA-AGENTS-INTERFACE{K, V} .
inc DATA-AGENTS-CONF .
inc CONFIGURATION .
pr MAP{K, V} * (sort Map{K, V} to Dict{K, V},
op _[_] to lookup,
op insert to update) .
pr SET{Request} .
vars A O C : Oid .
var D : Dict{K, V} .
var Key : K$Elt .
vars Val Val’ : [V$Elt] .
var Atts : AttributeSet .
var RS : Set{Request} .
*** class structure
op DataAgent : -> Cid [ctor] .
op data :_ : Dict{K, V} -> Attribute [ctor] .
op pal :_ : Oid -> Attribute [ctor] .
op requests :_ : Set{Request} -> Attribute [ctor] .
*** lookup request
rl [lookup] :
< A : DataAgent | data : D, pal : O, requests : RS >
msg(A, C, lookupReq(Key))
=> if lookup(D, Key) == undefined
then < A : DataAgent | data : D, pal : O,
requests : (RS, w4(O, C, lookupReq(Key))) >
msg(O, A, getReq(Key))
else < A : DataAgent | data : D, pal : O, requests : RS >
msg(C, A, lookupReply(Key, lookup(D, Key)))
fi .
*** lookup request missing data from pal
rl [getReq] :
< A : DataAgent | data : D, pal : O, Atts >
msg(A, O, getReq(Key))
=> < A : DataAgent | data : D, pal : O, Atts >
msg(O, A, getReply(Key, lookup(D, Key))) .
*** receive lookup requested missing data from pal

<!-- manual page 188 -->
176 CHAPTER 7. MODULE OPERATIONS
rl [getReply] :
< A : DataAgent | data : D, pal : O,
requests : (RS, w4(O, C, lookupReq(Key))) >
msg(A, O, getReply(Key, Val))
=> < A : DataAgent | pal : O, requests : RS,
data : if Val == undefined
then D
else update(Key, Val, D)
fi >
msg(C, A, lookupReply(Key, Val)) .
*** tell request
rl [tell] :
< A : DataAgent | data : D, requests : RS, pal : O >
msg(A, C, tellReq(Key, Val))
=> if lookup(D, Key) == undefined
then < A : DataAgent |
data : D,
requests : (RS, w4(O, C, tellReq(Key, Val))),
pal : O >
msg(O, A, setReq(Key, Val))
else < A : DataAgent | data : update(Key, Val, D),
requests : RS, pal : O >
msg(C, A, tellReply(Key, Val))
fi .
*** request update for missing data from pal
rl [setReq] :
< A : DataAgent | data : D, pal : O, Atts >
msg(A, O, setReq(Key, Val))
=> if lookup(D, Key) == undefined
then < A : DataAgent | data : D, pal : O, Atts >
msg(O, A, setReply(Key, undefined))
else < A : DataAgent | data : update(Key, Val, D),
pal : O, Atts >
msg(O, A, setReply(Key, Val))
fi .
*** receive requested update for missing data from pal
rl [setReply] :
< A : DataAgent | data : D, pal : O,
requests : (RS, w4(O, C, tellReq(Key, Val))) >
msg(A, O, setReply(Key, Val’))
=> < A : DataAgent | pal : O, requests : RS,
data : if Val’ == undefined
then update(Key, Val, D)
else D
fi >
msg(C, A, tellReply(Key, Val)) .
endm
The rule labeled lookup specifies how an agent handles a lookupReq message. The agent
first looks to see if its local dictionary contains the requested entry. If lookup(D, Key) ==
undefined, then a getReq is sent to the pal and the agent waits for a reply, remembering the
pending lookup request (w4(O, C, lookupReq(Key))). If the agent has the requested entry,

<!-- manual page 189 -->
7.5. EXAMPLE: DATA AGENTS 177
then it is returned in a lookupReply message.
The rules labeled getReq and getReply specify how agents exchange dictionary entries.
An agent can always answer a getReq message, since the Atts variable will match any status
attribute. The agent simply replies with the result, possibly undefined, of looking up the
requested key in its local dictionary. An agent only expects a getReply message if it has made
a request, and this only happens if the agent is trying to handle a lookupReq message. Thus
the rule only matches if the agent has the appropriate request w4(O, C, lookupReq(Key)) in
its requests attribute. The agent records the received reply with update(Key, Val, D) when
this reply is not undefined, and in any case sends it on to the customer with the message
msg(C, A, lookupReply(Key, Val)).
The rules labeled tell, setReq, and setReply specify how an agent handles a tellReq
message, following a protocol similar to the one described for the lookup request.
Note that in the case of agents with just these three attributes, using the Atts variable of
sort AttributeSet or the requests : RS expression, with RS a variable of sort Set{Request},
are equivalent ways of saying that the rule matches any set of requests. The first way is more
extensible, in that the rule would still work for agents belonging to a subclass of DataAgent
that uses additional attributes.
To test the data agent specification, we define a module AGENT-TEST. This module defines
object identifiers sam and joe for data agents, and me to name an external customer. It also
defines an initial configuration containing two agents named sam and joe with empty data
dictionaries, and two initial tellReq messages for each agent. We take both keys and data
elements to be quoted identifiers, by instantiating the parameterized DATA-AGENTS module
with the predefined Qid view.
mod AGENT-TEST is
ex DATA-AGENTS{Qid, Qid} .
ops sam joe me : -> Oid [ctor] .
op iconf : -> Configuration .
eq iconf
= < sam : DataAgent | data : empty,
pal : joe, requests : empty >
msg(sam, me, tellReq(’a, ’bc))
msg(sam, me, tellReq(’d, ’ef))
< joe : DataAgent | data : empty,
pal : sam, requests : empty >
msg(joe, me, tellReq(’g, ’hi))
msg(joe, me, tellReq(’j, ’kl)) .
endm
The importation graph of all the modules involved in this example is shown in Figure 7.3,
where the three different types of arrows correspond to the three different modes of importation
(protecting, extending, and including) used in the specification.
The following are results from test runs. First we rewrite the initial configuration iconf,
resulting in a configuration in which the agents have updated appropriately their data, and
there is one reply for each tellReq message.
Maude> rew iconf .
result Configuration:
< sam : DataAgent | data : (’a |-> ’bc, ’d |-> ’ef),
pal : joe, requests : empty >
< joe : DataAgent | data : (’g |-> ’hi, ’j |-> ’kl),
pal : sam, requests : empty >
msg(me, sam, tellReply(’a, ’bc))

<!-- manual page 190 -->
178 CHAPTER 7. MODULE OPERATIONS
AGENT-TEST
DATA-AGENTS{Qid, Qid}
MAP{Qid, Qid} * (γ) DATA-AGENTS-INTERFACE{Qid, Qid}
SET{Request}
DATA-AGENTS-CONF
QID
CONFIGURATION
Figure 7.3: Importation graph of data-agents modules
msg(me, sam, tellReply(’d, ’ef))
msg(me, joe, tellReply(’g, ’hi))
msg(me, joe, tellReply(’j, ’kl))
Next we try adding a lookup request and discover that, using Maude’s default rewriting
strategy, the lookup request is delivered before the tell requests, so the reply is undefined.
Maude> rew iconf msg(sam, me, lookupReq(’a)) .
result Configuration:
< sam : DataAgent | data : (’a |-> ’bc, ’d |-> ’ef),
pal : joe, requests : empty >
< joe : DataAgent | data : (’g |-> ’hi, ’j |-> ’kl),
pal : sam, requests : empty >
msg(me, sam, tellReply(’a, ’bc))
msg(me, sam, tellReply(’d, ’ef))
msg(me, sam, lookupReply(’a, undefined))
msg(me, joe, tellReply(’g, ’hi))
msg(me, joe, tellReply(’j, ’kl))
To see if a good answer can be obtained, we use the search command to look for a state in
which there is a lookupReply with data entry different from undefined.
Maude> search iconf msg(sam, me, lookupReq(’a))
=>! msg(me, sam, lookupReply(’a, Q:Qid)) C:Configuration .
Solution 1 (state 1081)
C:Configuration -->
< sam : DataAgent | data : (’a |-> ’bc, ’d |-> ’ef),
pal : joe, requests : empty >
< joe : DataAgent | data : (’g |-> ’hi, ’j |-> ’kl),
pal : sam, requests : empty >
msg(me, sam, tellReply(’a, ’bc))

<!-- manual page 191 -->
7.6. EXAMPLE: EXTENDED RENT-A-CAR STORE 179
msg(me, sam, tellReply(’d, ’ef))
msg(me, joe, tellReply(’g, ’hi))
msg(me, joe, tellReply(’j, ’kl))
Q:Qid --> ’bc
No more solutions.
Indeed, there is just one such reply.
Notice that two collaborating agents may get inconsistent data, that is, different values for
the same key, if they receive simultaneously tell requests for the same key. We may use the
search command to illustrate how this may happen.
Maude> search iconf
msg(sam, me, tellReq(’m, ’no))
msg(joe, me, tellReq(’m, ’pq))
=>! C:Configuration
< sam : DataAgent |
data : (’m |-> Q:Qid, R:Dict{Qid, Qid}),
Atts:AttributeSet >
< joe : DataAgent |
data : (’m |-> Q’:Qid, R’:Dict{Qid, Qid}),
Atts’:AttributeSet >
such that Q:Qid =/= Q’:Qid .
Solution 1 (state 5117)
C:Configuration -->
msg(me, sam, tellReply(’a, ’bc))
msg(me, sam, tellReply(’d, ’ef))
msg(me, sam, tellReply(’m, ’no))
msg(me, joe, tellReply(’g, ’hi))
msg(me, joe, tellReply(’j, ’kl))
msg(me, joe, tellReply(’m, ’pq))
Atts:AttributeSet --> pal : joe, requests : empty
R:Dict{Qid,Qid} --> ’a |-> ’bc, ’d |-> ’ef
Q:Qid --> ’no
Atts’:AttributeSet --> pal : sam, requests : empty
R’:Dict{Qid,Qid} --> ’g |-> ’hi, ’j |-> ’kl
Q’:Qid --> ’pq
No more solutions.
Note the use of the such that condition to filter search solutions (see Section 5.4.3 and A.4).
7.6 Example: extended rent-a-car store
This section describes a variant of the rent-a-car store example in Section 6.2 in which several
interesting data structures are used to store relevant information.
Let us refine the specification of a rent-a-car store presented in Section 6.2 by adding the
following regulations:
10. When a rented car is returned, the deposit is used to pay the rental charges, which are
calculated in accordance with the conditions at pick-up time.
11. There are three different kinds of customers: staff, occasional, and preferred.
12. Staff members and preferred customers benefit from special discounts in all rentals.

<!-- manual page 192 -->
180 CHAPTER 7. MODULE OPERATIONS
13. A customer qualifies as “preferred” when the accumulated amount of money spent in the
store by the customer is above a certain threshold.
The main differences introduced by these regulations are that we need to keep the conditions
atpick-uptime, sothatthecalculationsatdrop-offtimearecorrect. Wealsoneedtodistinguish
the three different types of customers, with the possibility of an occasional customer being
promoted to preferred if he spends a given amount of money.
As an alternative approach to the one followed previously in Section 6.2, we introduce a class
Store of rental car stores, whose attributes represent the information concerning the general
parameters of such stores: the rates applicable to each type of car, the discounts for each type
of customer renting each type of car, the identifiers of the customers who are suspended, the
amount of money above which occasional customers are qualified as preferred, the record with
the amount of money spent in the store by each of the customers, and the daily penalty for
late return (20%). In addition, attributes customers, cars, rentals, and calendar store the
identifiers of the objects participating in the relationships with the Store composite object;
those are directed binary relationships and therefore we need only store the identifiers of the
subordinate objects as attributes of the object that references them.
class Store |
discounts : PFun{Tuple{Cid, Cid}, Nat},
payments : PFun{Oid, Nat},
penalty : Nat,
threshold : Nat,
suspended : Set{Oid},
rates : PFun{Cid, Nat},
customers : Set{Oid},
cars : Set{Oid},
rentals : Set{Oid},
calendar : Oid .
The information on rates, discounts, and money spent is modeled by attributes of sort
PFun of partial functions9 (see Section 7.4.7), associating the appropriate values to each of the
different agents involved. The rates for the different cars are stored in the attribute rates, of
sort PFun{Cid, Nat}, that associates the per-day rate to be charged to a customer for renting
a given type of car. Thus, assuming that Rts is a variable of sort PFun{Cid, Nat}, with
value the partial function assigning the appropriate rates to each type of car, we have that
Rts[FullSizeCar] is the per-day rate for renting a full size car. If we want to increase this
rate by, say 20%, we can use the expression
Rts[FullSizeCar -> Rts[FullSizeCar] * (100 + penalty) / 100]
with penalty a constant equal to 20. The discounts applied to each customer on each type
of car and the amount of the purchases of each customer are stored, respectively, in attributes
payments and discounts. The set of the identifiers of the customers who are suspended is
stored in an attribute suspended of sort Set{Oid}. The predefined sorts Oid and Cid are used
for object identifiers and class identifiers, respectively.
This specification will allow us, for instance, to easily “compose” systems with different
particular details (e.g., discounts may change from one store to another), allowing them to
easily co-exist.
The rest of the classes can be specified as follows:
class Customer | cash : Nat, debt : Nat .
class Staff .
9An alternative possibility is to use the maps specified in the predefined MAP module in Section 8.15.

<!-- manual page 193 -->
7.6. EXAMPLE: EXTENDED RENT-A-CAR STORE 181
class OccasionalCust .
class PreferredCust .
subclasses OccasionalCust PreferredCust Staff < Customer .
class Car | available : Bool .
class EconomyCar .
class MidSizeCar .
class FullSizeCar .
subclasses EconomyCar MidSizeCar FullSizeCar < Car .
class Rental |
deposit : Nat, discount : Nat, dueDate : Nat, pickUpDate : Nat,
rate : Nat, customer : Oid, car : Oid .
The different actions may then be defined as follows:
crl [car-rental] :
< U : Customer | cash : M >
< I : Car | available : true > *** the car is available
< V : Store | suspended : US,
rates : Rts, discounts : Dscnts, calendar : C,
cars : (I, IS), customers : (U, SS), rentals : RS >
< C : Calendar | date : Today >
=> < U : Customer | cash : sd(M, Amnt) >
< I : Car | available : false >
< V : Store | rentals : (A, RS) >
< C : Calendar | >
< A : Rental | pickUpDate : Today, dueDate : Today + NumDays,
car : I, deposit : Amnt, customer : U,
rate : Rt, discount : Dscnt >
if not U in US *** the customer is not suspended
/\ Rt := Rts[getClass(< I : Car | >)]
/\ Dscnt := Dscnts[(getClass(< U : Customer | >),
getClass(< I : Car | >))]
/\ Amnt := sd(Rt, Dscnt) * NumDays
/\ M >= Amnt *** enough cash to make a deposit
[nonexec] .
Notice the use of customer and car attributes in objects of class Rental, which makes
explicit that a rental relationship is between the customer and the car specified by these at-
tributes. Likewise for attributes customers, cars, and calendar of object V of class Store,
which indicate that the customer, car, and calendar appearing on the rule should be known to
the store. After the car-rental action, the rental is added to the set of rentals kept by the
store.
Rules may be applied to objects of the classes specified in the rules or of any of their
subclasses. Remember that the function getClass takes an object as argument and returns its
actual class (see Section 6.1); for example, the getClass function applied to an object of the
form < ’c123 : MidSizeCar | ... > returns MidSizeCar, and not Car. Finally, notice the
use of matching equations of the form t := t’ in the condition (see Section 4.3).
The return of a rented car is specified by the rules below. The first rule handles the case in
which the car is returned on time, that is, the current date is smaller than or equal to the due
date, and therefore the deposit is greater than or equal to the amount due. Notice that the
rate and discount to be used in the calculation of the amount due are those at pick-up time,
which are stored as attributes of the Rental object.

<!-- manual page 194 -->
182 CHAPTER 7. MODULE OPERATIONS
crl [on-date-car-return] :
< U : Customer | cash : M >
< I : Car | >
< A : Rental | customer : U, car : I, rate : Rt, discount : Dscnt,
pickUpDate : Ppdt, dueDate : Ddt, deposit : Dpst >
< V : Store | payments : Pmnts, cars : (I, IS),
customers : (U, SS), calendar : C, rentals : (A, RS) >
< C : Calendar | date : Today >
=> < U : Customer | cash : M + sd(Dpst, Amnt) >
< I : Car | available : true >
< V : Store | rentals : RS,
payments : (if Pmnts[U] == undefined *** no record for customer
then Pmnts[U -> Amnt]
else Pmnts[U -> ((Pmnts[U]) + Amnt)]
fi) >
< C : Calendar | >
if (Today <= Ddt) /\ Amnt := sd(Rt, Dscnt) * sd(Today, Ppdt) .
In this case, the deposit is greater than the amount due and therefore part of the deposit needs
to be reimbursed. Note also that the Store object keeps a record of the amount of money
spent by each customer in the store, and thus it must be updated accordingly. We can see how
the Rental object disappears in the righthand side of the rules: it is removed from the set of
rentals known to the store and the availability of the car is restored.
The second rule deals with the case in which the car is returned late. The amount to be
paid is calculated at drop-off time, but the rate and discount to be used, those at pick-up time,
may have changed when returning the car.
crl [late-car-return] :
< U : Customer | debt : M >
< I : Car | >
< A : Rental | customer : U, car : I, rate : Rt, discount : Dscnt,
pickUpDate : Ppdt, dueDate : Ddt, deposit : Dpst >
< V : Store | payments : Pmnts, penalty : Pnlt, rentals : (A, RS),
cars : (I, IS), customers : (U, SS), calendar : C >
< C : Calendar | date : Today >
=> < U : Customer | debt : M + sd(Amnt, Dpst) >
< I : Car | available : true >
< V : Store | rentals : RS,
payments : (if Pmnts[U] == undefined
then Pmnts[U -> Dpst]
else Pmnts[U -> ((Pmnts[U]) + Dpst)]
fi) >
< C : Calendar | >
if Ddt < Today *** it is returned late
/\ Amnt := (sd(Rt, Dscnt) * sd(Ddt, Ppdt))
+ (((sd(Rt, Dscnt) * sd(Today, Ddt))
* (100 + Pnlt)) quo 100) .
In this case the customer’s debt is increased by the portion of the amount due not covered by
the deposit.
Debts may be paid at any time, the only condition being that the amount paid is between
zero and the amount of money of the customer at that time.
crl [pay-debt] :
< V : Store | payments : Pmnts, customers : (U, SS), calendar : C >

<!-- manual page 195 -->
7.6. EXAMPLE: EXTENDED RENT-A-CAR STORE 183
< U : Customer | debt : M, cash : N >
< C : Calendar | date : Today >
=> < V : Store | payments : Pmnts[U -> ((Pmnts[U]) + Amnt)] >
< U : Customer | debt : sd(M, Amnt), cash : sd(N, Amnt) >
< C : Calendar | >
if 0 < Amnt /\ Amnt <= N /\ Amnt <= M
[nonexec] .
We are assuming that, if there is a debt, then there has been a previous payment, and
therefore there is already a record for that customer.
The text says that customers who are late in returning a rented car or in paying their debts
“may” be suspended. However, nothing is said about the reasons for taking such a decision
or when they should be suspended, that is, a customer could be suspended right after the car
is returned without having paid all the charges, after some grace days, or never. In practice
there will be fixed criteria, as, for example, suspending customers that are two days late, or
two months.
The first rule deals with the case in which a customer has a pending debt, and the second
one handles the case in which a customer is late in returning a rented car.
crl [suspend-late-payers] :
< V : Store | suspended : US, customers : (U, SS) >
< U : Customer | debt : M >
=> < V : Store | suspended : (U, US) >
< U : Customer | >
if (not U in US) and M > 0 .
crl [suspend-late-returns] :
< V : Store | suspended : US, cars : (I, IS), customers : (U, SS), calendar : C >
< U : Customer | >
< I : Car | >
< A : Rental | customer : U, car : I, dueDate : F >
< C : Calendar | date : Today >
=> < V : Store | suspended : (U, US) >
< U : Customer | >
< I : Car | >
< A : Rental | >
< C : Calendar | >
if (not U in US) and F < Today .
The upgrade in the status of a customer can then be modeled with the following rule:
crl [upgrade-to-preferred] :
< U : OccasionalCust | cash : M, debt : N >
< V : St:Store | threshold : Thrshld, payments : Pmnts,
customers : (U, SS), calendar : C, Atts:AttributeSet >
< C : Ca:Calendar | date : Today, Atts’:AttributeSet >
=> < U : PreferredCust | cash : M, debt : N >
< V : St:Store | threshold : Thrshld, payments : Pmnts,
customers : (U, SS), calendar : C, Atts:AttributeSet >
< C : Ca:Calendar | date : Today, Atts’:AttributeSet >
if (Pmnts[U]) >= Thrshld
[dnt] .
Inthisrule,acustomerobjectUofsubclassOccasionalCustbecomesofsubclassPreferredCust
when the accumulated amount of purchases exceeds the store’s threshold. The partial function

<!-- manual page 196 -->
184 CHAPTER 7. MODULE OPERATIONS
stored in the attribute payments gives us the amount of money spent by each customer. Note
that since an object is changing it class, the rule must have the dntattribute (see Section 6.1.4).
Then, since the rule should be applied on objects of class OccasionalCust, but on objects of
classes Store and Calendar or their subclasses, we must explicitly use variables of respective
class types and add AttributeSet variables to gather non-specified attributes. Note that, in
addition to these AttributeSet variables, all attributes specified in the left-hand side must
also appear in the right-hand side. Otherwise they would disappear.
As in the simpler rent-a-car system, the presence of nonterminating rules and of rules with
new variables in the righthand side requires some kind of strategy for the execution of the
system; we give an example of such a strategy in Section 10.1.1.

