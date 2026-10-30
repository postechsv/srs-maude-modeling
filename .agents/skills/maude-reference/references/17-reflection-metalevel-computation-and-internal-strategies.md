# Chapter 17: Reflection, Metalevel Computation, and Internal Strategies (manual pages 431-488)

<!-- manual page 431 -->
Chapter 17
Reflection, Metalevel Computation,
and Internal Strategies
Informally, a reflective logic is a logic in which important aspects of its metatheory can be
represented at the object level in a consistent way, so that the object-level representation
correctly simulates the relevant metatheoretic aspects. In other words, a reflective logic is a
logic which can be faithfully interpreted in itself. Maude’s language design and implementation
make systematic use of the fact that rewriting logic is reflective [27, 20, 28, 29]. This makes the
metatheoryofrewritinglogicaccessibletotheuserinaclearandprincipledway. However, since
a naive implementation of reflection can be computationally expensive, a good implementation
must provide efficient ways of performing reflective computations. This chapter explains how
this is achieved in Maude through its predefined META-LEVEL module, that can be found in the
prelude.maude file.
17.1 Reflection and metalevel computation
Rewriting logic is reflective in a precise mathematical way, namely, there is a finitely presented
rewrite theory Uthat is universal in the sense that we can represent in Uany finitely presented
rewrite theory R(including Uitself) as a term R, any terms t,t′ in Ras terms t,t′, and any
pair (R,t) as a term ⟨R,t⟩, in such a way that we have the following equivalence
R⊢t−→t′ ⇔ U⊢⟨R,t⟩−→⟨R,t′⟩.
SinceUisrepresentableinitself, wecanachievea“reflectivetower” withanarbitrarynumber
of levels of reflection:
R⊢t→t′ ⇔ U⊢⟨R,t⟩→⟨R,t′⟩ ⇔ U⊢⟨U,⟨R,t⟩⟩→⟨U,⟨R,t′⟩⟩...
In this chain of equivalences we say that the first rewriting computation takes place at level
0, the second at level 1, and so on. In a naive implementation, each step up the reflective tower
comes at considerable computational cost, because simulating a single step of rewriting at one
level involves many rewriting steps one level up. It is therefore important to have systematic
ways of lowering the levels of reflective computations as much as possible, so that a rewriting
subcomputation happens at a higher level in the tower only when this is strictly necessary.
InMaude, keyfunctionalityoftheuniversaltheoryUhasbeenefficientlyimplementedinthe
functional module META-LEVEL. This module includes the modules META-VIEW, META-MODULE,
META-STRATEGY, and META-TERM. As an overview,
419

<!-- manual page 432 -->
420CHAPTER 17. REFLECTION, METALEVEL COMPUTATION, AND INTERNAL STRATEGIES
• in the module META-TERM, Maude terms are metarepresented as elements of a data type
Term of terms;
• inthemoduleMETA-STRATEGY,theexpressionsoftheMaudestrategylanguagearemetarep-
resented as terms in a data type Strategy of strategy expressions;
• in the module META-MODULE, Maude modules are metarepresented as terms in a data type
Module of modules;
• in the module META-VIEW, Maude views are metarepresented as terms in a data type View
of views; and
• in the module META-LEVEL,
– operations upModule, upTerm, downTerm, and others allow moving between reflection
levels;
– the process of reducing a term to canonical form using Maude’s reduce command is
metarepresented by a built-in function metaReduce;
– the processes of rewriting a term in a system module using Maude’s rewrite and
frewrite commands are metarepresented by built-in functions metaRewrite and
metaFrewrite;
– the process of applying (without extension) a rule of a system module at the top of
a term is metarepresented by a built-in function metaApply;
– the process of applying (with extension) a rule of a system module at any position
of a term is metarepresented by a built-in function metaXapply;
– the process of matching (without extension) two terms at the top is reified by a
built-in function metaMatch;
– the process of matching (with extension) a pattern to any subterm of a term is reified
by a built-in function metaXmatch;
– the process of searching for a term satisfying some conditions starting in an initial
term is reified by built-in functions metaSearch and metaSearchPath;
– theprocessesofrewritingatermusingMaude’ssrewriteanddsrewritecommands
are metarepresented by the built-in function metaSrewrite;
– the process of unifying two terms modulo axioms is reified by built-in functions
metaUnify, metaIrredundantUnify, metaDisjointUnify and metaIrredundant-
DisjointUnify;
– theprocessofreducingatermtoallitscanonicalformsusingMaude’sget variants
command is metarepresented by a built-in function metaGetVariant;
– the process of unifying (resp. matching) two terms modulo axioms and variant equa-
tions is reified by built-in functions metaVariantUnify and metaVariantDisjoint-
Unify (resp. metaVariantMatch);
– the process of symbolic searching for a term in an initial term is reified by built-in
functionsmetaNarrowingApply,metaNarrowingSearchandmetaNarrowingSearch-
Path;
– checking satisfiability of a given formula is reified by built-in function metaCheck;
and
– parsing and pretty-printing of terms and strategy expressions in a module, as well as
key sort operations such as comparing sorts in the subsort ordering of a signature,
are also metarepresented by corresponding built-in functions.
ThefunctionsmetaReduce, metaApply, metaXapply, metaRewrite, metaFrewrite, metaMatch,
metaXmatch, and metaSrewrite are called descent functions, since they allow us to descend
levels in the reflective tower. The paper [23] provides a formal definition of the notion of
descent function, and a detailed explanation of how they can be used to achieve a systematic,
conservative way of lowering the levels of reflective computations.

<!-- manual page 433 -->
17.2. THE META-TERM MODULE 421
META-LEVEL
META-VIEW
META-MODULE
QID-LIST QID-SET*(β)
META-STRATEGY
NAT-LIST
_*(β)
QID-SET
QID
META-TERM
META-CONDITION
Figure 17.1: Importation graph of metalevel modules
The importation graph in Figure 17.1 shows the relationships between all the modules in the
metalevel. The modules NAT-LIST and QID-LIST provide lists of natural numbers and quoted
identifiers, respectively (see Section 8.14.1), and the module QID-SET provides sets of quoted
identifiers (see Section 8.14.2). Notice that QID-SET is imported (in protecting mode) with
renaming
(op empty to none, op _,_ to _;_ [prec 43])
abbreviated to β in the figure.
17.2 The META-TERM module
17.2.1 Metarepresenting sorts and kinds
In the META-TERM module, sorts and kinds are metarepresented as data in specific subsorts of
the sort Qid of quoted identifiers.
A term of sort Sort is any quoted identifier not containing the following characters: ‘:’, ‘.’,
‘[’, and ‘]’. Moreover, the characters ‘{’, ‘}’, and ‘,’ can only appear in structured sort names
(see Section 3.3). For example, ’Bool, ’NzNat, ’a‘{X‘}, ’a‘{X‘,Y‘}, ’a‘{b‘,c‘{d‘}‘}‘{e‘},
and ’a‘{‘(‘} are terms of sort Sort.
An element of sort Kind is a quoted identifier of the form ’‘[SortList‘] where SortList is
a single identifier formed by a list of unquoted elements of sort Sort separated by backquoted
commas. For example, ’‘[Bool‘] and ’‘[NzNat‘,Zero‘,Nat‘] are valid elements of the sort
Kind. Note the use of backquotes to force them to be single identifiers.

<!-- manual page 434 -->
422CHAPTER 17. REFLECTION, METALEVEL COMPUTATION, AND INTERNAL STRATEGIES
Since commas and square brackets are used to metarepresent kinds, these characters are
forbidden in sort names, in order to avoid undesirable ambiguities. Periods and colons are also
forbidden, due to the metarepresentation of constants and variables, as explained in the next
section.
Since operator declarations can use both sorts and kinds, we denote by Type the union of
Sort and Kind.
sorts Sort Kind Type .
subsorts Sort Kind < Type < Qid.
op <Qids> : -> Sort [special (...)] .
op <Qids> : -> Kind [special (...)] .
Remember from the introduction of Chapter 8 that <Qids> is a special operator declaration
used to represent sets of constants that are not algebraically constructed, but are instead
associated with appropriate C++ code by “hooks” which are specified following the special
attribute; see the functional module META-TERM in file prelude.maude for the details omitted
here.
17.2.2 Metarepresenting terms
In the module META-TERM, terms are metarepresented as elements of the data type Term of
terms. The base cases in the metarepresentation of terms are given by subsorts Constant and
Variable of the sort Qid.
sorts Constant Variable Term .
subsorts Constant Variable < Qid Term .
op <Qids> : -> Constant [special (...)] .
op <Qids> : -> Variable [special (...)] .
Constants are quoted identifiers that contain the constant’s name and its type separated by
a ‘.’, e.g., ’0.Nat. Similarly, variables contain their name and type separated by a ‘:’, e.g.,
’N:Nat. Appropriate selectors then extract their names and types.
op getName : Constant -> Qid .
op getName : Variable -> Qid .
op getType : Constant -> Type .
op getType : Variable -> Type .
Since ‘.’ and ‘:’ are not allowed in sort names (see Section 3.3), the name and type of a
constant or variable can be calculated easily. Note that there is no restriction in operator or in
variable names, and thus the scanning for ‘.’ or ‘:’ is done from right to left in the identifier.
That is,
getName(’:-D:Smile) = ’:-D
getType(’:-.|.‘[Smile‘]) = ’‘[Smile‘]
A term different from a constant or a variable is constructed in the usual way, by applying
an operator symbol to a nonempty list of terms.
sorts NeTermList TermList .
subsorts Term < NeTermList < TermList .
op _,_ : TermList TermList -> TermList
[ctor assoc id: empty gather (e E) prec 121] .
op _,_ : NeTermList TermList -> NeTermList [ctor ditto] .
op _,_ : TermList NeTermList -> NeTermList [ctor ditto] .
op _[_] : Qid NeTermList -> Term [ctor] .

<!-- manual page 435 -->
17.3. THE META-STRATEGY MODULE: METAREPRESENTING THE STRATEGY LANGUAGE423
The actual sort infrastructure provided by the module META-TERM is a bit more complex,
because there are also subsorts and operators for the metarepresentation of ground terms and
thecorrespondinglistsofgroundtermsthatwedonotdescribehere(seethefileprelude.maude
for details).
Since terms in the module META-TERM can be metarepresented just as terms in any other
module, the metarepresentation of terms can be iterated.
For example, the term c q M:Marking in the module VENDING-MACHINE in Section 5.1 is
metarepresented by
’__[’c.Item, ’__[’q.Coin, ’M:Marking]]
and meta-metarepresented by
’_‘[_‘][’’__.Qid,
’_‘,_[’’c.Item.Constant,
’_‘[_‘][’’__.Qid,
’_‘,_[’’q.Coin.Constant,
’’M:Marking.Variable]]]]
Note that the metarepresentation of a natural number such as, e.g., 42 is ’s_^42[’0.Zero]
instead of ’42.NzNat, since, as explained in Section 8.3, 42is just syntactic sugar for s_^42(0).
17.3 TheMETA-STRATEGY module: Metarepresentingthestrat-
egy language
All components of the strategy language described in Section 10, including its modules and
views, can be manipulated at the metalevel. The META-STRATEGY module metarepresents all
the strategy combinators as terms of the sort Strategy, with two subsorts, RuleApplication
for rule applications, and CallStrategy for strategy calls.
ops fail idle : -> Strategy [ctor] .
op all : -> RuleApplication [ctor] .
op _[_]{_} : Qid Substitution StrategyList -> RuleApplication [ctor prec 21] .
op top : RuleApplication -> Strategy [ctor] .
op match_s.t._ : Term EqCondition -> Strategy [ctor prec 21] .
op xmatch_s.t._ : Term EqCondition -> Strategy [ctor prec 21] .
op amatch_s.t._ : Term EqCondition -> Strategy [ctor prec 21] .
op _|_ : Strategy Strategy -> Strategy
[ctor assoc comm id: fail prec 41 gather (e E)] .
op _;_ : Strategy Strategy -> Strategy [ctor assoc id: idle prec 39 gather (e E)] .
op _or-else_ : Strategy Strategy -> Strategy [ctor assoc prec 43 gather (e E)] .
op _+ : Strategy -> Strategy [ctor] .
op _?_:_ : Strategy Strategy Strategy -> Strategy [ctor prec 55] .
op matchrew_s.t._by_ : Term EqCondition UsingPairSet -> Strategy [ctor] .
op xmatchrew_s.t._by_ : Term EqCondition UsingPairSet -> Strategy [ctor] .
op amatchrew_s.t._by_ : Term EqCondition UsingPairSet -> Strategy [ctor] .
op _[[_]] : Qid TermList -> CallStrategy [ctor prec 21] .
op one : Strategy -> Strategy [ctor] .
The metarepresentation of strategy expressions is very similar to their object-level syntax, ex-
cept that strategy calls are written with double square brackets instead of parentheses. Strategy
lists and variable-strategy pair sets, required for the rule application and matchrew operators
respectively, are also represented as terms.

<!-- manual page 436 -->
424CHAPTER 17. REFLECTION, METALEVEL COMPUTATION, AND INTERNAL STRATEGIES
op empty : -> StrategyList [ctor] .
op _,_ : StrategyList StrategyList -> StrategyList [ctor assoc id: empty] .
op _using_ : Variable Strategy -> UsingPair [ctor prec 21] .
op _,_ : UsingPairSet UsingPairSet -> UsingPairSet [ctor assoc comm prec 61] .
eq U:UsingPair, U:UsingPair = U:UsingPair .
Conditions are described in the module META-CONDITION included by META-STRATEGY.
fmod META-CONDITION is
protecting META-TERM .
sorts EqCondition Condition .
subsort EqCondition < Condition .
op nil : -> EqCondition [ctor] .
op _=_ : Term Term -> EqCondition [ctor prec 71] .
op _:_ : Term Sort -> EqCondition [ctor prec 71] .
op _:=_ : Term Term -> EqCondition [ctor prec 71] .
op _=>_ : Term Term -> Condition [ctor prec 71] .
op _/\_ : EqCondition EqCondition -> EqCondition
[ctor assoc id: nil prec 73] .
op _/\_ : Condition Condition -> Condition
[ctor assoc id: nil prec 73] .
endfm
The derived strategy combinators, explained in the last part of Section 10.1, are also defined
as constructors. For efficiency purposes they are encoded internally in a special form, different
from the encoding of their equivalent expressions.
op _or-else_ : Strategy Strategy -> Strategy [ctor assoc prec 43 gather (e E)] .
op _* : Strategy -> Strategy [ctor] .
op _! : Strategy -> Strategy [ctor] .
op not : Strategy -> Strategy [ctor] .
op test : Strategy -> Strategy [ctor] .
op try : Strategy -> Strategy [ctor] .
For example, the metarepresentation of the strategy one(move * ; amatch (0)[nil]) for
the HANOI example in Section 10 is
one(’move[none]{empty} * ; amatch ’‘(_‘)‘[_‘][’0.Zero, ’nil.NatList] s.t. nil)
17.4 The META-MODULE module: Metarepresenting modules
In the module META-MODULE, which imports META-TERM and META-STRATEGY, functional, system
and strategy modules, as well as functional, system and strategy theories, are metarepresented
in a syntax very similar to their original user syntax.
The main differences are that:
1. terms in equations, membership axioms, and rules are now metarepresented as we have
already explained in Section 17.2.2;
2. in the metarepresentation of modules and theories we follow a fixed order in introducing
the different kinds of declarations for sorts, subsort relations, equations, etc., whereas in
the user syntax there is considerable flexibility for introducing such different declarations
in an interleaved and piecemeal way;
3. there is no need for variable declarations—in fact, there is no syntax for metarepresenting
them—and

<!-- manual page 437 -->
17.4. THE META-MODULE MODULE: METAREPRESENTING MODULES 425
4. sets of identifiers—used in declarations of sorts—are metarepresented as sets of quoted
identifiers built with an associative and commutative operator _;_.
The syntax for the top-level operators metarepresenting modules and theories is as follows,
where Header means just an identifier in the case of non-parameterized modules or an identifier
together with a list of parameter declarations in the case of a parameterized module.
sorts FModule SModule StratModule FTheory STheory StratTheory Module .
subsorts FModule < SModule < StratModule < Module .
subsorts FTheory < STheory < StratTheory < Module .
sort Header .
subsort Qid < Header .
op _{_} : Qid ParameterDeclList -> Header [ctor] .
op fmod_is_sorts_.____endfm : Header ImportList SortSet
SubsortDeclSet OpDeclSet MembAxSet EquationSet -> FModule
[ctor gather (& & & & & & &)] .
op mod_is_sorts_._____endm : Header ImportList SortSet
SubsortDeclSet OpDeclSet MembAxSet EquationSet RuleSet
-> SModule [ctor gather (& & & & & & & &)] .
op smod_is_sorts_._______endsm : Header ImportList SortSet
SubsortDeclSet OpDeclSet MembAxSet EquationSet RuleSet
StratDeclSet StratDefSet -> StratModule
[ctor gather (& & & & & & & & & &)] .
op fth_is_sorts_.____endfth : Qid ImportList SortSet SubsortDeclSet
OpDeclSet MembAxSet EquationSet -> FTheory
[ctor gather (& & & & & & &)] .
op th_is_sorts_._____endth : Qid ImportList SortSet SubsortDeclSet
OpDeclSet MembAxSet EquationSet RuleSet -> STheory
[ctor gather (& & & & & & & &)] .
op sth_is_sorts_._______endsth : Qid ImportList SortSet
SubsortDeclSet OpDeclSet MembAxSet EquationSet RuleSet
StratDeclSet StratDefSet -> StratTheory
[ctor gather (& & & & & & & & & &)] .
Appropriate selectors then extract from the metarepresentation of modules the metarepresen-
tations of their names, imported submodules, and declared sorts, subsorts, operators, member-
ships, equations, rules, strategy declarations, and strategy definitions.
op getName : Module -> Qid .
op getImports : Module -> ImportList .
op getSorts : Module -> SortSet .
op getSubsorts : Module -> SubsortDeclSet .
op getOps : Module -> OpDeclSet .
op getMbs : Module -> MembAxSet .
op getEqs : Module -> EquationSet .
op getRls : Module -> RuleSet .
op getStrats : Module -> StratDeclSet .
op getSds : Module -> StratDefSet .
Without going into all the syntactic details, we show only the operators used to metarep-
resent sets of sorts and kinds, equations, rules and strategies. The complete syntax used for
metarepresenting modules can be found in the module META-MODULE in the file prelude.maude.
Conditions are defined in the module META-CONDITION shown in Section 17.3.
sorts EmptyTypeSet NeSortSet NeKindSet
NeTypeSet SortSet KindSet TypeSet .
subsort EmptyTypeSet < SortSet KindSet < TypeSet < QidSet .

<!-- manual page 438 -->
426CHAPTER 17. REFLECTION, METALEVEL COMPUTATION, AND INTERNAL STRATEGIES
subsort Sort < NeSortSet < SortSet .
subsort Kind < NeKindSet < KindSet .
subsort Type NeSortSet NeKindSet < NeTypeSet < TypeSet NeQidSet .
op none : -> EmptyTypeSet [ctor] .
op _;_ : TypeSet TypeSet -> TypeSet
[ctor assoc comm id: none prec 43] .
op _;_ : SortSet SortSet -> SortSet [ctor ditto] .
op _;_ : KindSet KindSet -> KindSet [ctor ditto] .
sorts Equation EquationSet .
subsort Equation < EquationSet .
op eq_=_[_]. : Term Term AttrSet -> Equation [ctor] .
op ceq_=_if_[_]. : Term Term EqCondition AttrSet -> Equation
[ctor] .
op none : -> EquationSet [ctor] .
op __ : EquationSet EquationSet -> EquationSet
[ctor assoc comm id: none] .
sorts Rule RuleSet .
subsort Rule < RuleSet .
op rl_=>_[_]. : Term Term AttrSet -> Rule [ctor] .
op crl_=>_if_[_]. : Term Term Condition AttrSet -> Rule [ctor] .
op none : -> RuleSet [ctor] .
op __ : RuleSet RuleSet -> RuleSet [ctor assoc comm id: none] .
sorts StratDecl StratDeclSet .
subsort StratDecl < StratDeclSet .
op strat_:_@_[_]. : Qid TypeList Type AttrSet -> StratDecl [ctor] .
op none : -> StratDeclSet [ctor] .
op __ : StratDeclSet StratDeclSet -> StratDeclSet [ctor assoc comm id: none] .
sorts StratDefinition StratDefSet .
subsort StratDefinition < StratDefSet .
op sd_:=_[_]. : CallStrategy Strategy AttrSet -> StratDefinition [ctor] .
op csd_:=_if_[_]. : CallStrategy Strategy EqCondition AttrSet
-> StratDefinition [ctor] .
op none : -> StratDefSet [ctor] .
op __ : StratDefSet StratDefSet -> StratDefSet [ctor assoc comm id: none] .
For example, we show here the metarepresentations of the modules introduced in Section 5.1
VENDING-MACHINE-SIGNATURE and VENDING-MACHINE.
fmod ’VENDING-MACHINE-SIGNATURE is
nil
sorts ’Coin ; ’Item ; ’Marking .
subsort ’Coin < ’Marking .
subsort ’Item < ’Marking .
op ’__ : ’Marking ’Marking -> ’Marking
[assoc comm id(’null.Marking)] .
op ’a : nil -> ’Item [format(’b! ’o)] .
op ’null : nil -> ’Marking [none] .
op ’$ : nil -> ’Coin [format(’r! ’o)] .
op ’q : nil -> ’Coin [format(’r! ’o)] .
op ’c : nil -> ’Item [format(’b! ’o)] .
none

<!-- manual page 439 -->
17.4. THE META-MODULE MODULE: METAREPRESENTING MODULES 427
none
endfm
mod ’VENDING-MACHINE is
including ’VENDING-MACHINE-SIGNATURE .
sorts none .
none
none
none
none
rl ’M:Marking => ’__[’M:Marking, ’q.Coin] [label(’add-q)] .
rl ’M:Marking => ’__[’M:Marking, ’$.Coin] [label(’add-$)] .
rl ’$.Coin => ’c.Item [label(’buy-c)] .
rl ’$.Coin => ’__[’a.Item, ’q.Coin] [label(’buy-a)] .
rl ’__[’q.Coin, ’__[’q.Coin, ’__[’q.Coin, ’q.Coin]]]
=> ’$.Coin [label(’change)] .
endm
Since VENDING-MACHINE-SIGNATURE has no list of imported submodules, no membership
axioms, and no equations, those fields are filled, respectively, with the constants nil of sort
ImportList, none of sort MembAxSet, and none of sort EquationSet. Similarly, since the mod-
ule VENDING-MACHINE has no subsort declarations and no operator declarations, those fields are
filled,respectively,withtheconstantsnoneofsortSubsortDeclSetandnoneofsortOpDeclSet.
Variable declarations are not metarepresented, but rather each variable is metarepresented in
its “on the fly”-declaration form, i.e., with its sort or kind.
As mentioned above, parameterized modules are also metarepresented through the notion of
a header, which is either an identifier (for non-parameterized modules) or an identifier together
with a list of parameter declarations (for parameterized modules). Such parameter declarations
are metarepresented again with a syntax similar to the user syntax.
sorts ParameterDecl NeParameterDeclList ParameterDeclList .
subsorts ParameterDecl < NeParameterDeclList < ParameterDeclList .
op _::_ : Sort ModuleExpression -> ParameterDecl .
op nil : -> ParameterDeclList [ctor] .
op _,_ : ParameterDeclList ParameterDeclList -> ParameterDeclList
[ctor assoc id: nil prec 121] .
Module expressions involving renamings and summations can also be metarepresented with
the expected syntax:
sort ModuleExpression .
subsort Qid < ModuleExpression .
op _+_ : ModuleExpression ModuleExpression -> ModuleExpression
[ctor assoc comm] .
op _*(_) : ModuleExpression RenamingSet -> ModuleExpression
[ctor prec 39 format (d d s n++i n--i d)] .
sorts Renaming RenamingSet .
subsort Renaming < RenamingSet .
op sort_to_ : Qid Qid -> Renaming [ctor] .
op op_to_[_] : Qid Qid AttrSet -> Renaming
[ctor format (d d d d s d d d)] .
op op_:_->_to_[_] : Qid TypeList Type Qid AttrSet -> Renaming
[ctor format (d d d d d d d d s d d d)] .
op label_to_ : Qid Qid -> Renaming [ctor] .

<!-- manual page 440 -->
428CHAPTER 17. REFLECTION, METALEVEL COMPUTATION, AND INTERNAL STRATEGIES
op strat_to_ : Qid Qid -> Renaming [ctor] .
op strat_:_@_to_ : Qid TypeList Type Qid -> Renaming [ctor] .
op _,_ : RenamingSet RenamingSet -> RenamingSet
[ctor assoc comm prec 43 format (d d ni d)] .
Finally, the instantiation of a parameterized module is metarepresented as follows:
op _{_} : ModuleExpression ParameterList -> ModuleExpression
[ctor prec 37] .
sort EmptyCommaList NeParameterList ParameterList .
subsorts Sort < NeParameterList < ParameterList .
subsort EmptyCommaList < GroundTermList ParameterList .
op empty : -> EmptyCommaList [ctor] .
op _,_ : ParameterList ParameterList -> ParameterList [ctor ditto] .
The rules for constructing parameterized metamodules and instantiating parameterized
modules existing in the database reflect the object-level rules. In particular, bound param-
eters are permitted; for example, the following term metarepresents a parameterized module:
fmod ’PARMODEX{’X :: ’TRIV} is
including ’MAP{’String, ’X} .
sorts ’Foo .
none
none
none
none
endfm
Although, as we will see in the following section, views can be metarepresented as terms
of the View sort, it is not possible to use the views constructed at the metalevel in module
expressions. The views used in the module expressions occurring in metamodules must have
been declared at the object level, so that they are present in the database of modules and views
declared in the given session. However, when using meta-interpreters, explained in Section 19,
views constructed at the metalevel can be added to their databases and then used in mod-
ule expressions. Such views are written in quoted form within metamodule expressions, like
’String in ’MAP{’String, ’X} in the example above.
Note that terms of sort Module can be metarepresented again, yielding then a term of sort
Term, and this can be iterated an arbitrary number of times. This is in fact necessary when a
metalevel computation has to operate at higher levels.
17.5 The META-VIEW module: Metarepresenting views
In the module META-VIEW, which imports META-MODULE, views are metarepresented in a syntax
very similar to their original user syntax.
sort View .
op view_from_to_is___endv : Header ModuleExpression ModuleExpression
SortMappingSet OpMappingSet StratMappingSet -> View [ctor gather (& & & & & &)
format (d d d d d d d n++i ni ni n--i d)] .
The first argument corresponds to the header of sort Header of the view, which is just an
identifier or an identifier together with a list of parameter declarations depending on whether
the view is a parameterized one, like in the case of modules. The second and third are module
expressions corresponding to the source (usually a theory) and target (usually a module) of the

<!-- manual page 441 -->
17.5. THE META-VIEW MODULE: METAREPRESENTING VIEWS 429
view, respectively. The fourth, fifth and sixth arguments are the sort, operator and strategy
mappings defining the view.
The following syntax defines sets of sort mappings in a way completely similar to the user
syntax.
sorts SortMapping SortMappingSet .
subsort SortMapping < SortMappingSet .
op sort_to_. : Sort Sort -> SortMapping [ctor] .
op none : -> SortMappingSet [ctor] .
op __ : SortMappingSet SortMappingSet -> SortMappingSet
[ctor assoc comm id: none format (d ni d)] .
eq S:SortMapping S:SortMapping = S:SortMapping .
Analogously, the following syntax is used to define set of operator mappings and strategy
mappings.
sorts OpMapping OpMappingSet .
subsort OpMapping < OpMappingSet .
op (op_to_.) : Qid Qid -> OpMapping [ctor] .
op (op_:_->_to_.) : Qid TypeList Type Qid -> OpMapping [ctor] .
op (op_to term_.) : Term Term -> OpMapping [ctor] .
op none : -> OpMappingSet [ctor] .
op __ : OpMappingSet OpMappingSet -> OpMappingSet
[ctor assoc comm id: none format (d ni d)] .
eq O:OpMapping O:OpMapping = O:OpMapping .
sorts StratMapping StratMappingSet .
subsort StratMapping < StratMappingSet .
op (strat_to_.) : Qid Qid -> StratMapping [ctor] .
op (strat_:_@_to_.) : Qid TypeList Type Qid -> StratMapping [ctor] .
op (strat_to-expr_.) : CallStrategy Strategy -> StratMapping [ctor] .
op none : -> StratMappingSet [ctor] .
op __ : StratMappingSet StratMappingSet -> StratMappingSet
[ctor assoc comm id: none format (d ni d)] .
eq S:StratMapping S:StratMapping = S:StratMapping .
Finally, appropriate selectors are used to extract from the metarepresentation of a view the
corresponding components, namely, the metarepresentations of its name, of its source, of its
target, of its set of sort mappings, its set of operator mappings, and its set of strategy mappings.
op getName : View -> Qid .
op getFrom : View -> ModuleExpression .
op getTo : View -> ModuleExpression .
op getSortMappings : View -> SortMappingSet .
op getOpMappings : View -> OpMappingSet .
op getStratMappings : View -> StratMappingSet .
For example, the metarepresentation of the view RingToRat (see Section 7.4.2) from the
theory RING to the predefined RAT module is as follows:
view ’RingToRat from ’RING to ’RAT is
sort ’Ring to ’Rat .
op ’z to ’0 .
op ’e.Ring to term ’s_[’0.Zero] .
none
endv

<!-- manual page 442 -->
430CHAPTER 17. REFLECTION, METALEVEL COMPUTATION, AND INTERNAL STRATEGIES
Then, we can extract some components of this metarepresented view:
Maude> reduce in META-VIEW :
getFrom(view ’RingToRat from ’RING to ’RAT is
sort ’Ring to ’Rat .
op ’z to ’0 .
op ’e.Ring to term ’s_[’0.Zero] .
none
endv) .
result Sort: ’RING
Maude> reduce in META-VIEW :
getOpMappings(view ’RingToRat from ’RING to ’RAT is
sort ’Ring to ’Rat .
op ’z to ’0 .
op ’e.Ring to term ’s_[’0.Zero] .
none
endv) .
result OpMappingSet:
op ’z to ’0 .
op ’e.Ring to term ’s_[’0.Zero] .
17.6 The META-LEVEL module: Metalevel operations
The META-LEVEL module, which imports META-VIEW, has several built-in descent functions that
provide useful and efficient ways of reducing metalevel computations to object-level ones, as
well as several useful operations on sorts and kinds. Since, in general, these operations take
among their arguments the metarepresentations of modules, sorts, kinds, terms, and so on, the
META-LEVEL modules also provides several built-in functions for moving conveniently between
reflection levels. Notice that most of the operations in the module META-LEVEL are partial (as
explicitly stated by using the arrow ~> in the corresponding operator declaration). This is due
to the fact that they do not make sense on terms that, although may be of the correct sort,
for example, Module or Term, either are not correct metarepresentations of modules or are not
correct metarepresentations of terms in the module provided as another argument.
Concerning partial operations, the criteria used to choose between using a supersort for the
result and having an operator map to a kind is as follows.
If the error return value is built from constructors, say
op noParse : Nat -> ResultPair? [ctor] .
op ambiguity : ResultPair ResultPair -> ResultPair? [ctor] .
it goes to a supersort. In some sense these are not errors, but merely exceptions or out-of-band
results for which there is a carefully defined semantics.
The kind is reserved for nonconstructors which may not be able to reduce at all on illegal
arguments, like, for example, in the function (notice the form of the arrow)
op metaParse : Module VariableSet QidList Type? ~> ResultPair? [special (...)] .
In this second case, an expression that does not evaluate to the appropriate sort represents a
real error.
So, for example, a call to metaParsewith an ill-formed module would produce an unreduced
term metaParse(...) in the kind, whereas a call to metaParse with valid arguments but a
list of tokens that could not be parsed to a term of the desired type in the metamodule would
produce a term noParse(...) of sort ResultPair? indicating where the parse first failed.

<!-- manual page 443 -->
17.6. THE META-LEVEL MODULE: METALEVEL OPERATIONS 431
17.6.1 Moving between reflection levels: upModule, upTerm, downTerm,
and others
ForamoduleRthathas alreadybeenloaded intoMaude,theoperationsupSorts,upSubsortDecl,
upOpDecls, upMbs, upEqs, upRls, upStratDecls, upSds, and upModule take as arguments the
metarepresentation of the name of Rand a Boolean value b, and return, respectively, the
metarepresentations of the module R, of its sorts, subsort declarations, operator declarations,
membership axioms, equations, rules, strategy declarations, and strategy definitions. If the
second argument of these functions is true, then the resulting metarepresentations will include
the corresponding statements that Rimports from its submodules; but if the second argument
is false, the resulting metarepresentations will only contain the metarepresentations of the
statements explicitly declared in R.
op upModule : Qid Bool ~> Module [special (...)] .
op upSorts : Qid Bool ~> SortSet [special (...)] .
op upSubsortDecls : Qid Bool ~> SubsortDeclSet [special (...)] .
op upOpDecls : Qid Bool ~> OpDeclSet [special (...)] .
op upMbs : Qid Bool ~> MembAxSet [special (...)] .
op upEqs : Qid Bool ~> EquationSet [special (...)] .
op upRls : Qid Bool ~> RuleSet [special (...)] .
op upStratDecls : Qid Bool ~> StratDeclSet [special (...)] .
op upSds : Qid Bool ~> StratDefSet [special (...)] .
We give below simple examples of using these functions. Note that, since BOOL is automati-
cally imported by all modules, its equations are shown when upEqsis called with trueas its sec-
ondargument. Forthesamereason,themetarepresentationoftheVENDING-MACHINE-SIGNATURE
module includes an including declaration that was not explicit in that module. Here, and in
the rest of this section, we assume that the modules NUMBERS and SIEVE from Chapter 4, as
well as the modules VENDING-MACHINE-SIGNATURE and VENDING-MACHINE from Chapter 5, have
already been loaded into Maude.
Maude> reduce in META-LEVEL :
upModule(’VENDING-MACHINE-SIGNATURE, false) .
result FModule:
fmod ’VENDING-MACHINE-SIGNATURE is
including ’BOOL .
sorts ’Coin ; ’Item ; ’Marking .
subsort ’Coin < ’Marking .
subsort ’Item < ’Marking .
op ’$ : nil -> ’Coin [format(’r! ’o)] .
op ’__ : ’Marking ’Marking -> ’Marking
[assoc comm id(’null.Marking)] .
op ’a : nil -> ’Item [format(’b! ’o)] .
op ’c : nil -> ’Item [format(’b! ’o)] .
op ’null : nil -> ’Marking [none] .
op ’q : nil -> ’Coin [format(’r! ’o)] .
none
none
endfm
Maude> reduce in META-LEVEL : upEqs(’VENDING-MACHINE, true) .
result EquationSet:
eq ’_and_[’true.Bool, ’A:Bool] = ’A:Bool [none] .
eq ’_and_[’A:Bool, ’A:Bool] = ’A:Bool [none] .

<!-- manual page 444 -->
432CHAPTER 17. REFLECTION, METALEVEL COMPUTATION, AND INTERNAL STRATEGIES
eq ’_and_[’A:Bool, ’_xor_[’B:Bool, ’C:Bool]]
= ’_xor_[’_and_[’A:Bool, ’B:Bool], ’_and_[’A:Bool, ’C:Bool]]
[none] .
eq ’_and_[’false.Bool, ’A:Bool] = ’false.Bool [none] .
eq ’_or_[’A:Bool,’B:Bool]
= ’_xor_[’_and_[’A:Bool, ’B:Bool],’_xor_[’A:Bool, ’B:Bool]]
[none] .
eq ’_xor_[’A:Bool, ’A:Bool] = ’false.Bool [none] .
eq ’_xor_[’false.Bool, ’A:Bool] = ’A:Bool [none] .
eq ’not_[’A:Bool] = ’_xor_[’true.Bool, ’A:Bool] [none] .
eq ’_implies_[’A:Bool, ’B:Bool]
= ’not_[’_xor_[’A:Bool, ’_and_[’A:Bool, ’B:Bool]]] [none] .
Maude> reduce in META-LEVEL : upEqs(’VENDING-MACHINE, false) .
result EquationSet: (none).EquationSet
Maude> reduce in META-LEVEL : upRls(’VENDING-MACHINE, true) .
result RuleSet:
rl ’$.Coin => ’c.Item [label(’buy-c)] .
rl ’$.Coin => ’__[’q.Coin,’a.Item] [label(’buy-a)] .
rl ’M:Marking => ’__[’$.Coin,’M:Marking] [label(’add-$)] .
rl ’M:Marking => ’__[’q.Coin,’M:Marking] [label(’add-q)] .
rl ’__[’q.Coin,’q.Coin,’q.Coin,’q.Coin] => ’$.Coin
[label(’change)] .
In addition to the upModule operator, there is another operator allowing the use of an
already loaded module at the metalevel. This operator is defined in the module META-MODULE
as follows:
op [_] : Qid -> Module .
eq [Q:Qid] = (sth Q:Qid is including Q:Qid .
sorts none . none none none none none none none endsth) .
This operator is just syntactic sugar for accessing the corresponding module. Notice that
the module is not moved up to the metalevel as upModule does, it is just a way of referring to
it, and therefore more efficient.
The META-LEVEL module also provides a function upImports that takes as argument the
metarepresentation of the name of a module R. When Ris already in the Maude module
database, then upImports returns the metarepresentation of its list of imported submodules.
The function upImports does not take a Boolean argument, as the previous up-functions, since
it is not useful to ask for the list of imported submodules of a flattened module.
op upImports : Qid ~> ImportList [special (...)] .
In the same way, the META-LEVELmodule provides a function upViewthat takes as argument
the metarepresentation of the name of a view; when such a view is in the Maude view database,
then upView returns the corresponding metarepresentation.
op upView : Qid ~> View [special (...)] .
As a simple example, let us consider the view String0 from the predefined theory DEFAULT
to the predefined module STRING, all of them provided in prelude.maude; then,
Maude> reduce in META-LEVEL : upView(’String0) .
result View:
view ’String0 from ’DEFAULT to ’STRING is

<!-- manual page 445 -->
17.6. THE META-LEVEL MODULE: METALEVEL OPERATIONS 433
sort ’Elt to ’String .
op ’0.Elt to term ’"".String .
endv
Finally,theMETA-LEVELmoduleintroducestwopolymorphicfunctions. ThefunctionupTerm
takes a term tand returns the metarepresentation of its canonical form. The function downTerm
takes the metarepresentation of a term t as its first argument and a term t′ as its second ar-
gument, and returns the canonical form of t, if t is a term in the same kind as t′; otherwise, it
returns the canonical form of t′
.
op upTerm : Universal -> Term [poly (1) special (...)] .
op downTerm : Term Universal -> Universal
[poly (2 0) special (...)] .
As simple examples, we can use the function upTerm to obtain the metarepresentation of
the term f(a, f(b, c)) in the module UP-DOWN-TEST below, and the function downTerm to
recover the term f(a, f(b, c)) from its metarepresentation.
fmod UP-DOWN-TEST is
protecting META-LEVEL .
sort Foo .
ops a b c d : -> Foo .
op f : Foo Foo -> Foo .
op error : -> [Foo] .
eq c = d .
endfm
Maude> reduce in UP-DOWN-TEST : upTerm(f(a, f(b, c))) .
result GroundTerm: ’f[’a.Foo,’f[’b.Foo,’d.Foo]]
Notice in the previous example that the given argument has been reduced before obtaining
itsmetarepresentation, morespecifically, thesubtermchasbecomed. Inthefollowingexamples
we can observe the same behavior with respect to downTerm.
Maude> reduce in UP-DOWN-TEST :
downTerm(’f[’a.Foo,’f[’b.Foo,’c.Foo]], error) .
result Foo: f(a, f(b, d))
Maude> reduce in UP-DOWN-TEST :
downTerm(upTerm(f(a, f(b, c))), error) .
result Foo: f(a, f(b, d))
In our last example, we show the result of downTerm when its first argument does not cor-
respond to the metarepresentation of a term in the module UP-DOWN-TEST; notice the constant
e in the metarepresented term that does not correspond to a declared constant in the module.
Maude> reduce in UP-DOWN-TEST :
downTerm(’f[’a.Foo,’f[’b.Foo,’e.Foo]], error) .
Advisory: could not find a constant e of
sort Foo in meta-module UP-DOWN-TEST.
result [Foo]: error
Due to the failure in moving down the metarepresented term given as first argument, the
result is the term given as second argument, namely, error, which was declared in the module
UP-DOWN-TEST as a constant of kind [Foo].

<!-- manual page 446 -->
434CHAPTER 17. REFLECTION, METALEVEL COMPUTATION, AND INTERNAL STRATEGIES
17.6.2 Simplifying: metaReduce and metaNormalize
metaReduce
The (partial) operation metaReduce takes as arguments the metarepresentation of a module R
and the metarepresentation of a term t.
sort ResultPair .
op {_,_} : Term Type -> ResultPair [ctor] .
op metaReduce : Module Term ~> ResultPair [special (...)] .
When t is a term in R, metaReduce(R,t) returns the metarepresentation of the canonical
form of t, using the equations in R, together with the metarepresentation of its corresponding
sort or kind. The reduction strategy used by metaReduce coincides with that of the reduce
command (see Sections 4.9 and A.2).
As said above, in general, when either the first argument of metaReduce is a term of sort
Module but not a correct metarepresentation Rof an object module R, or the second argument
is not the correct metarepresentation tof a term tin R, the operation metaReduce is undefined,
that is, the term metaReduce(u,v) does not reduce and it does not get evaluated to a term of
sort ResultPair, but only to an expression in the kind [ResultPair].
Appropriate selectors extract from the result pairs their two components:
op getTerm : ResultPair -> Term .
op getType : ResultPair -> Type .
Using metaReduce we can simulate at the metalevel the primes computation example at the
end of Section 4.4.7.
Maude> reduce in META-LEVEL :
metaReduce(upModule(’SIEVE, false),
’show_upto_[’primes.NatList, ’s_^10[’0.Zero]]) .
result ResultPair:
{’_._[’s_^2[’0.Zero], ’s_^3[’0.Zero], ’s_^5[’0.Zero],
’s_^7[’0.Zero], ’s_^11[’0.Zero], ’s_^13[’0.Zero],
’s_^17[’0.Zero], ’s_^19[’0.Zero], ’s_^23[’0.Zero],
’s_^29[’0.Zero]],
’NatList}
We can also insert a new element into an empty map of the type declared in the module
PARMODEX at the end of Section 17.4 as follows:
Maude> red in META-LEVEL :
metaReduce(
fmod ’PARMODEX{’X :: ’TRIV} is
including ’MAP{’String, ’X} .
sorts ’Foo .
none
none
none
none
endfm,
’insert[’"foo".String, ’A:X$Elt,
’empty.Map‘{String‘,X‘}]) .
result ResultPair:
{’_|->_[’"foo".String,’A:X$Elt],’Entry‘{String‘,X‘}}
Noticethatthemoduleexpression’MAP{’String, ’X}hasaboundparameterX,whichappears
also in the sort X$Elt in the on-the-fly declaration of the variable A:X$Elt.

<!-- manual page 447 -->
17.6. THE META-LEVEL MODULE: METALEVEL OPERATIONS 435
metaNormalize
The (partial) operation metaNormalize takes as arguments the metarepresentation of a module
Rand the metarepresentation of a term t.
op metaNormalize : Module Term ~> ResultPair [special (...)] .
When tis a term in R, metaNormalize(R,t) returns the metarepresentation of the normal
form of t with respect to the equational theory consisting of the equational attributes of the
operators in t, without doing any simplification or rewriting with respect to equations or rules
in R, together with the metarepresentation of its corresponding sort or kind. For example,
from the declarations in the predefined NAT module
op s_ : Nat -> NzNat [ctor iter special (...)] .
op _+_ : NzNat Nat -> NzNat [assoc comm prec 33 special (...)] .
op _+_ : Nat Nat -> Nat [ditto] .
we know that the successor operator satisfies the iter theory (see Section 4.4.2) and that the
addition operator is associative and commutative (see Section 4.4.1). With this information it
is easy to make sense of the following results:
Maude> red in META-LEVEL :
metaNormalize(upModule(’NAT, false), ’s_[’s_[’0.Zero]]) .
result ResultPair: {’s_^2[’0.Zero],’NzNat}
Maude> red in META-LEVEL :
metaNormalize(upModule(’NAT, false),
’_+_[’s_[’s_[’0.Zero]],’0.Zero]) .
result ResultPair: {’_+_[’0.Zero,’s_^2[’0.Zero]],’NzNat}
Maude> red in META-LEVEL :
metaNormalize(upModule(’NAT, false),
’_+_[’0.Zero,’_+_[’s_[’s_[’0.Zero]],’0.Zero]]) .
result ResultPair: {’_+_[’0.Zero,’0.Zero,’s_^2[’0.Zero]],’NzNat}
Notice that associative terms are flattened and, if they are also commutative, the subterms are
sorted with respect to an internal order. Notice also that in the last two examples the subterm
’0.Zero does not disappear. This is because 0 is not declared as an identity element for _+_.
17.6.3 Rewriting: metaRewrite and metaFrewrite
metaRewrite
The (partial) operation metaRewrite takes as arguments the metarepresentation of a module
R, the metarepresentation of a term t, and a value b of the sort Bound, i.e., either a natural
number or the constant unbounded.
sort Bound .
subsort Nat < Bound .
op unbounded :-> Bound [ctor] .
op metaRewrite : Module Term Bound ~> ResultPair [special (...)] .
The operation metaRewrite is entirely analogous to metaReduce, but instead of using only
theequationalpartofamoduleitnowusesboththeequationsandtherulestorewritetheterm.
The reduction strategy used by metaRewrite coincides with that of the rewrite command (see
Sections 5.4 and A.2). That is, the result of metaRewrite(R, t, b) is the metarepresentation

<!-- manual page 448 -->
436CHAPTER 17. REFLECTION, METALEVEL COMPUTATION, AND INTERNAL STRATEGIES
of the term obtained from t after at most b applications of the rules in Rusing the rewrite
strategy, togetherwiththemetarepresentationofitscorrespondingsortorkind. Whenthevalue
unbounded is given as the third argument, no bound is imposed to the number of rewrites, and
rewriting proceeds to the bitter end.
Using metaRewrite we can redo at the metalevel the examples in Section 5.4.
Maude> reduce in META-LEVEL :
metaRewrite(upModule(’VENDING-MACHINE, false),
’__[’$.Coin, ’__[’$.Coin, ’__[’q.Coin, ’q.Coin]]], 1) .
result ResultPair:
{’__[’$.Coin, ’$.Coin, ’q.Coin, ’q.Coin, ’q.Coin], ’Marking}
Maude> reduce in META-LEVEL :
metaRewrite(upModule(’VENDING-MACHINE, false),
’__[’$.Coin, ’__[’$.Coin, ’__[’q.Coin, ’q.Coin]]], 2) .
result ResultPair:
{’__[’$.Coin, ’$.Coin, ’$.Coin, ’q.Coin, ’q.Coin, ’q.Coin],
’Marking}
metaFrewrite
Position-fair rewriting, which was described in Section 5.4, is metarepresented by the operation
metaFrewrite. This(partial)operationtakesasargumentsthemetarepresentationofamodule,
the metarepresentation of a term, a value of sort Bound, and a natural number.
op metaFrewrite : Module Term Bound Nat ~> ResultPair
[special (...)] .
The reduction strategy used by metaFrewrite coincides with that of the frewrite command
in Maude, except that a final (semantic) sort calculation is performed at the end in order to
produce a correct ResultPair. That is, frewrite(R, t, b, n) results in the metarepresen-
tation of the term obtained from t after at most b applications of the rules in Rusing the
frewrite strategy, with at most n rewrites at each entitled position on each traversal of a
subject term, together with the metarepresentation of its corresponding sort or kind. When
the value unbounded is given as the third argument, no bound is imposed to the number of
rewrites.
Using metaFrewrite we can redo at the metalevel the examples in Section 5.4.
Maude> reduce in META-LEVEL :
metaFrewrite(upModule(’VENDING-MACHINE, false),
’__[’$.Coin, ’__[’$.Coin, ’__[’q.Coin, ’q.Coin]]],
1, 1) .
result ResultPair:
{’__[’$.Coin, ’q.Coin, ’q.Coin, ’c.Item], ’Marking}
Maude> reduce in META-LEVEL :
metaFrewrite(upModule(’VENDING-MACHINE, false),
’__[’$.Coin, ’__[’$.Coin, ’__[’q.Coin, ’q.Coin]]],
12, 1) .
result ResultPair:
{’__[’$.Coin, ’$.Coin, ’$.Coin, ’$.Coin, ’q.Coin, ’q.Coin,
’q.Coin, ’q.Coin, ’q.Coin, ’q.Coin, ’q.Coin, ’q.Coin,
’q.Coin,’a.Item,’c.Item],
’Marking}

<!-- manual page 449 -->
17.6. THE META-LEVEL MODULE: METALEVEL OPERATIONS 437
17.6.4 Applying rules: metaApply and metaXapply
metaApply
The (partial) operation metaApply takes as arguments the metarepresentation of a module, the
metarepresentation of a term, the metarepresentation of a rule label, the metarepresentation
of a set of assignments (possibly empty) defining a partial substitution, and a natural number.
sorts Assignment Substitution .
subsort Assignment < Substitution .
op _<-_ : Variable Term -> Assignment [ctor prec 63] .
op none : -> Substitution [ctor] .
op _;_ : Substitution Substitution -> Substitution
[assoc comm id: none prec 65] .
sort ResultTriple ResultTriple? .
subsort ResultTriple < ResultTriple? .
op {_,_,_} : Term Type Substitution -> ResultTriple [ctor] .
op failure : -> ResultTriple? [ctor] .
op metaApply : Module Term Qid Substitution Nat ~> ResultTriple?
[special (...)] .
The operation metaApply(R, t, l, σ, n) is evaluated as follows:
1. the term t is first fully reduced using the equations in R;
2. the resulting term is matched at the top against all rules with label l in Rpartially
instantiated with σ, with matches that fail to satisfy the condition of their rule discarded;
3. the first n successful matches are discarded; if there is an (n+ 1)th match, its rule is
applied using that match and the steps 4 and 5 below are taken; otherwise failure is
returned;
4. the term resulting from applying the given rule with the (n+ 1)th match is fully reduced
using the equations in R;
5. the triple formed by the metarepresentation of the resulting fully reduced term, the
metarepresentation of its corresponding sort or kind, and the metarepresentation of the
substitution used in the reduction is returned.
The failure value should not be confused with the “undefined” value for the metaApply
operation. Asalreadymentionedbeforefordescentfunctionsingeneral, thisoperationispartial
because it does not make sense on some nonvalid arguments that are terms of the appropriate
sort but are not correct metarepresentations. However, even if all arguments are valid in this
sense, the intended rule application may fail, either because there is no match or because the
match does not satisfy the corresponding rule condition, and then failure is used to represent
this situation, which is important to distinguish from ill-formed invocations, for example, for
error recovery purposes.
Notealsothat, accordingtotheinformationinstep3above, thelastargumentof metaApply
isanaturalnumberusedtoenumerate(startingfrom0)allthepossiblesolutionsoftheintended
rule application. For efficiency, the different solutions should be generated in order, that is,
starting with the argument 0 and increasing it until a failure is obtained, indicating that there
are no more solutions.
Appropriate selectors extract from the result triples their three components:
op getTerm : ResultTriple -> Term .
op getType : ResultTriple -> Type .
op getSubstitution : ResultTriple -> Substitution .

<!-- manual page 450 -->
438CHAPTER 17. REFLECTION, METALEVEL COMPUTATION, AND INTERNAL STRATEGIES
As an example, we can force at the metalevel the rewriting of the term $ in the module
VENDING-MACHINE, so that only the rule buy-c is used, and only once.
Maude> reduce in META-LEVEL :
metaApply(upModule(’VENDING-MACHINE, false),
’$.Coin, ’buy-c, none, 0) .
result ResultTriple: {’c.Item, ’Item, none}
Similarly, we can force the rewriting of the same term so that this time only the rule add-$
is applied.
Maude> reduce in META-LEVEL :
metaApply(upModule(’VENDING-MACHINE, false),
’$.Coin, ’add-$, none, 0) .
result ResultTriple:
{’__[’$.Coin, ’$.Coin], ’Marking, ’M:Marking <- ’$.Coin}
However, using metaApply, we cannot force the term q $ to be rewritten with the rule
buy-c, since its lefthand side, $, does not match (without extension) this term. In this case,
we should use instead the metaXapply operation described below.
Maude> reduce in META-LEVEL :
metaApply(upModule(’VENDING-MACHINE, false),
’__[’q.Coin, ’$.Coin], ’buy-c, none, 0) .
result ResultTriple?: (failure).ResultTriple?
metaXapply
The (partial) operation metaXapply takes as arguments the metarepresentation of a module,
themetarepresentationofaterm, themetarepresentationofarulelabel, themetarepresentation
of a set of assignments (possibly empty) defining a partial substitution, a natural number, a
Bound value, and another natural number.
TheoperationmetaXapply(R, t, l, σ, n, b, m)isevaluatedasthefunctionmetaApply
but using extension (see Section 4.8) and in any possible position, not only at the top. The
arguments n and b can be used to localize the part of the term where the rule application can
take place:
• n is the lower bound on depth in terms of nested operators, and should be set to 0 to
start searching from the top, while
• the Bound argument bindicates the upper bound, and should be set to unbounded to have
no cut off.
Notice that nested occurrences of an operator with the assoc attribute are counted as a
single operator for depth purposes, that is, matching takes place on the flattened term (see
Section 4.8). The same idea applies to iter operators (see section 4.4.2): a whole stack of an
iter operator counts as a single operator. Furthermore, because of matching with extension,
the solution may have an extra layer, as illustrated in the matching examples at the end of
Section 17.6.5.
The last Nat argument m in metaXapply(R, t, l, σ, n, b, m), as in the case of the
operation metaApply, is the solution number, used to enumerate multiple solutions. The first
solution is 0, and they should again be generated in order for efficiency.
The result of metaXapply has an additional component, giving the context (a term with a
single “hole”, represented []) inside the given term t, where the rewriting has taken place. The
sort NeCTermList represents nonempty lists of terms with exactly one “hole,” that is, exactly
one term of sort Context, the rest being of sort Term. The sort GTermList is the supersort of
NeCTermListandTermListneededfortheassocattribute(hiddeninthefollowingdeclarations
in the ditto attribute) to make sense.

<!-- manual page 451 -->
17.6. THE META-LEVEL MODULE: METALEVEL OPERATIONS 439
sorts Context NeCTermList GTermList .
subsorts Context < NeCTermList < GTermList .
subsort TermList < GTermList .
op [] : -> Context [ctor] .
op _,_ : TermList NeCTermList -> NeCTermList [ctor ditto] .
op _,_ : NeCTermList TermList -> NeCTermList [ctor ditto] .
op _,_ : GTermList GTermList -> GTermList [ctor ditto] .
op _[_] : Qid NeCTermList -> Context [ctor] .
sorts Result4Tuple Result4Tuple? .
subsort Result4Tuple < Result4Tuple? .
op {_,_,_,_} : Term Type Substitution Context -> Result4Tuple
[ctor] .
op failure : -> Result4Tuple? [ctor] .
op metaXapply :
Module Term Qid Substitution Nat Bound Nat ~> Result4Tuple?
[special (...)] .
Appropriate selectors extract from the result 4-tuples their four components:
op getTerm : Result4Tuple -> Term .
op getType : Result4Tuple -> Type .
op getSubstitution : Result4Tuple -> Substitution .
op getContext : Result4Tuple -> Context .
As an example, we can force at the metalevel the rewriting of the term $ q in the mod-
ule VENDING-MACHINE so that only the rule buy-c is used (compare with the last metaApply
example).
Maude> reduce in META-LEVEL :
metaXapply(upModule(’VENDING-MACHINE, false),
’__[’q.Coin, ’$.Coin], ’buy-c, none, 0, unbounded, 0) .
result Result4Tuple:
{’__[’q.Coin, ’c.Item], ’Marking, none, ’__[’q.Coin, []]}
Notice the fragment ’__[’q.Coin, []] of the result, providing the context where the rule
has been applied. Since this is the only possible solution, if we request the “next” solution (by
increasing to 1 the last argument), the result will be a failure.
Maude> reduce in META-LEVEL :
metaXapply(upModule(’VENDING-MACHINE, false),
’__[’q.Coin, ’$.Coin], ’buy-c, none, 0, unbounded, 1) .
result Result4Tuple?: (failure).Result4Tuple?
17.6.5 Matching: metaMatch and metaXmatch
The (partial) operation metaMatch takes as arguments the metarepresentation of a module,
the metarepresentations of two terms, the metarepresentation of a condition, and a natural
number.
sort Substitution? .
subsort Substitution < Substitution? .
op noMatch : -> Substitution? [ctor] .
op metaMatch : Module Term Term Condition Nat ~> Substitution?
[special (...)] .

<!-- manual page 452 -->
440CHAPTER 17. REFLECTION, METALEVEL COMPUTATION, AND INTERNAL STRATEGIES
The operation metaMatch(R, t, t′
, Cond, n) tries to match at the top the terms t and
t′ in the module Rin such a way that the resulting substitution satisfies the condition Cond.
The last argument is used to enumerate possible matches. If the matching attempt is successful,
the result is the corresponding substitution; otherwise, noMatch is returned. The generalization
to metaXmatch follows exactly the same ideas as for metaXapply. Notice that the operation
metaMatch provides the metalevel counterpart of the object-level command match, while the
operation metaXmatch provides a generalization of the object-level command xmatch (see Sec-
tions 4.7, 4.8, and A.3) in that it is possible to specify min and max depths (in terms of
theory layers) and search for proper subterms that do not belong to the top theory layer. The
object-level behavior of the xmatch command is obtained by setting both min and max depth
to 0.
sorts MatchPair MatchPair? .
subsort MatchPair < MatchPair? .
op {_,_} : Substitution Context -> MatchPair [ctor] .
op noMatch : -> MatchPair? [ctor] .
op metaXmatch :
Module Term Term Condition Nat Bound Nat ~> MatchPair?
[special (...)] .
Appropriate selectors extract from the result pairs their two components:
op getSubstitution : MatchPair -> Substitution .
op getContext : MatchPair -> Context .
In the following examples, we try to match the pattern M:Marking $ with the term $ q c
a in several different ways:
• at the top, asking for the first solution,
Maude> reduce in META-LEVEL :
metaMatch(upModule(’VENDING-MACHINE, false),
’__[’M:Marking, ’$.Coin],
’__[’$.Coin, ’q.Coin, ’a.Item, ’c.Item],
nil, 0) .
result Assignment:
’M:Marking <- ’__[’q.Coin, ’a.Item, ’c.Item]
• at the top, asking for the second solution (that does not exist in this example)
Maude> reduce metaMatch(upModule(’VENDING-MACHINE, false),
’__[’M:Marking, ’$.Coin],
’__[’$.Coin, ’q.Coin, ’a.Item, ’c.Item],
nil, 1) .
result Substitution?: (noMatch).Substitution?
• anywhere, asking for the first solution,
Maude> reduce metaXmatch(upModule(’VENDING-MACHINE, false),
’__[’M:Marking, ’$.Coin],
’__[’$.Coin, ’q.Coin, ’a.Item, ’c.Item],
nil, 0, unbounded, 0) .
result MatchPair:
{’M:Marking <- ’__[’q.Coin, ’a.Item, ’c.Item], []}
• anywhere, asking for the second solution,
Maude> reduce metaXmatch(upModule(’VENDING-MACHINE, false),
’__[’M:Marking, ’$.Coin],

<!-- manual page 453 -->
17.6. THE META-LEVEL MODULE: METALEVEL OPERATIONS 441
’__[’$.Coin, ’q.Coin, ’a.Item, ’c.Item],
nil, 0, unbounded, 1) .
result MatchPair:
{’M:Marking <- ’__[’a.Item, ’c.Item], ’__[’q.Coin, []]}
• at the top, asking for the first solution satisfying a given condition (that again does not
exist),
Maude> reduce metaMatch(upModule(’VENDING-MACHINE, false),
’__[’M:Marking, ’$.Coin],
’__[’$.Coin, ’q.Coin, ’a.Item, ’c.Item],
’M:Marking = ’a.Item, 0) .
result Substitution?: (noMatch).Substitution?
• anywhere, asking for the first solution satisfying a given condition,
Maude> reduce metaXmatch(upModule(’VENDING-MACHINE, false),
’__[’M:Marking, ’$.Coin],
’__[’$.Coin, ’q.Coin, ’a.Item, ’c.Item],
’M:Marking = ’a.Item, 0, unbounded, 0) .
result MatchPair:
{’M:Marking <- ’a.Item, ’__[’__[’q.Coin, ’c.Item], []]}
As mentioned in the previous section, when matching with extension, the solution may have
an extra layer. Let us consider, for example, the following module:
fmod METAXMATCH-EX is
pr META-LEVEL .
op foo : QidSet ~> QidSet .
endfm
Then we take at the metalevel the pattern _;_(’A, QS:QidSet) and the (flattened) subject
term foo(_;_(’A, ’B, ’C)), and ask for matches with extension under at most 1 theory layer,
as shown in the following reductions:
Maude> red metaXmatch(upModule(’METAXMATCH-EX, false),
upTerm((’A ; QS:QidSet)),
upTerm(foo(’A ; ’B ; ’C)), nil, 0, 1, 0) .
result MatchPair: {’QS:QidSet <- ’_;_[’’B.Sort, ’’C.Sort], ’foo[[]]}
Maude> red metaXmatch(upModule(’METAXMATCH-EX, false),
upTerm((’A ; QS:QidSet)),
upTerm(foo(’A ; ’B ; ’C)), nil, 0, 1, 1) .
result MatchPair: {’QS:QidSet <- ’’C.Sort, ’foo[’_;_[’’B.Sort, []]]}
Maude> red metaXmatch(upModule(’METAXMATCH-EX, false),
upTerm((’A ; QS:QidSet)),
upTerm(foo(’A ; ’B ; ’C)), nil, 0, 1, 2) .
result MatchPair: {’QS:QidSet <- ’’B.Sort, ’foo[’_;_[’’C.Sort, []]]}
Maude> red metaXmatch(upModule(’METAXMATCH-EX, false),
upTerm((’A ; QS:QidSet)),
upTerm(foo(’A ; ’B ; ’C)), nil, 0, 1, 3) .
result MatchPair?: (noMatch).MatchPair?
Obviously, there is no match at the top, but under one theory layer (the foo operator) we
have _;_(’A, ’B, ’C). The first solution is the expected one, with the variable QS:QidSet

<!-- manual page 454 -->
442CHAPTER 17. REFLECTION, METALEVEL COMPUTATION, AND INTERNAL STRATEGIES
matching the subterm _;_(’B, ’C). However, in the next two solutions we see that we also
have the variable QS:QidSet matching either the fragment ’C or ’B while the other fragment
goes into the extension. Then the context in the solution has 2 theory layers but this is just a
feature of matching with extension: some solutions will have an extra layer.
As another example of this situation, let us consider the following reductions:
Maude> reduce in META-LEVEL :
metaXmatch(upModule(’METAXMATCH-EX, false),
upTerm(s N:Nat), upTerm(prec(s_^2(0))), nil, 0, 1, 0) .
result MatchPair: {’N:Nat <- ’s_[’0.Zero], ’prec[[]]}
Maude> red metaXmatch(upModule(’METAXMATCH-EX, false),
upTerm(s N:Nat), upTerm(prec(s_^2(0))), nil, 0, 1, 1) .
result MatchPair: {’N:Nat <- ’0.Zero, ’prec[’s_[[]]]}
Here the context in the first solution has one theory layer while the context in the second
has two, but the actual matching problem solved (with extension), namely, s N <=? s_^2(0)
under the single theory layer provided by the operator prec is the same in both reductions.
17.6.6 Searching: metaSearch and metaSearchPath
metaSearch
The operation metaSearch takes as arguments the metarepresentation of a module, the meta-
representation of the starting term for search, the metarepresentation of the pattern to search
for, the metarepresentation of a condition to be satisfied, the metarepresentation of the kind of
search to carry on, a Bound value, and a natural number.
op metaSearch :
Module Term Term Condition Qid Bound Nat ~> ResultTriple?
[special (...)] .
The searching strategy used by metaSearch coincides with that of the object-level search
command in Maude (see Sections 5.4 and A.4). The Qid values that are allowed as arguments
are: ’* for a search involving zero or more rewrites (corresponding to =>* in the search
command), ’+ for a search consisting in one or more rewrites (=>+), and ’! for a search that
only matches canonical forms (=>!). The Bound argument indicates the maximum depth of the
search, and the Nat argument is the solution number. To indicate a search consisting in exactly
one rewrite, we set the maximum depth of the search to the number 1.
Using metaSearch we can redo at the metalevel the last example in Section 5.4. The results
give us the answer to the question: if I have already inserted one dollar and three quarters in
the vending machine, can I get two cakes and an apple? The answer is yes; in fact, there are
several ways.
Maude> reduce in META-LEVEL :
metaSearch(upModule(’VENDING-MACHINE, false),
’__[’$.Coin, ’q.Coin, ’q.Coin,’q.Coin],
’__[’c.Item, ’a.Item, ’c.Item, ’M:Marking],
nil, ’+, unbounded, 0) .
result ResultTriple:
{’__[’q.Coin,’q.Coin,’q.Coin,’q.Coin,’a.Item,’c.Item,’c.Item],
’Marking,
’M:Marking <- ’__[’q.Coin, ’q.Coin, ’q.Coin, ’q.Coin]}

<!-- manual page 455 -->
17.6. THE META-LEVEL MODULE: METALEVEL OPERATIONS 443
Maude> reduce in META-LEVEL :
metaSearch(upModule(’VENDING-MACHINE, false),
’__[’$.Coin, ’q.Coin, ’q.Coin, ’q.Coin],
’__[’c.Item, ’a.Item, ’c.Item, ’M:Marking],
nil, ’+, unbounded, 1) .
result ResultTriple:
{’__[’a.Item, ’c.Item, ’c.Item],
’Marking,
’M:Marking <- ’null.Marking}
metaSearchPath
The operation metaSearchPath is complementary to metaSearch and carries out the same
search, butinsteadofreturningthefinalstateandmatchingsubstitutionitreturnsthesequence
of states and rules on a path starting with the reduced initial state and leading to (but not
including) the final state.
op metaSearchPath :
Module Term Term Condition Qid Bound Nat ~> Trace?
[special (...)] .
The sort Trace is used to represent the path as a list of triples by means of the following
syntax:
sorts TraceStep Trace Trace? .
subsorts TraceStep < Trace < Trace? .
op {_,_,_} : Term Type Rule -> TraceStep [ctor] .
op nil : -> Trace [ctor] .
op __ : Trace Trace -> Trace [ctor assoc id: nil format (d n d)] .
op failure : -> Trace? [ctor] .
We run again the same two examples as above, with the following results.
Maude> reduce in META-LEVEL :
metaSearchPath(upModule(’VENDING-MACHINE, false),
’__[’$.Coin, ’q.Coin, ’q.Coin,’q.Coin],
’__[’c.Item, ’a.Item, ’c.Item, ’M:Marking],
nil, ’+, unbounded, 0) .
result Trace:
{’__[’$.Coin,’q.Coin,’q.Coin,’q.Coin],
’Marking,
rl ’M:Marking => ’__[’$.Coin,’M:Marking] [label(’add-$)] .}
{’__[’$.Coin,’$.Coin,’q.Coin,’q.Coin,’q.Coin],
’Marking,
rl ’M:Marking => ’__[’$.Coin,’M:Marking] [label(’add-$)] .}
{’__[’$.Coin,’$.Coin,’$.Coin,’q.Coin,’q.Coin,’q.Coin],
’Marking,
rl ’$.Coin => ’c.Item [label(’buy-c)] .}
{’__[’$.Coin,’$.Coin,’q.Coin,’q.Coin,’q.Coin,’c.Item],
’Marking,
rl ’$.Coin => ’c.Item [label(’buy-c)] .}
{’__[’$.Coin,’q.Coin,’q.Coin,’q.Coin,’c.Item,’c.Item],
’Marking,
rl ’$.Coin => ’__[’q.Coin,’a.Item] [label(’buy-a)] .}

<!-- manual page 456 -->
444CHAPTER 17. REFLECTION, METALEVEL COMPUTATION, AND INTERNAL STRATEGIES
Maude> reduce in META-LEVEL :
metaSearchPath(upModule(’VENDING-MACHINE, false),
’__[’$.Coin, ’q.Coin, ’q.Coin, ’q.Coin],
’__[’c.Item, ’a.Item, ’c.Item, ’M:Marking],
nil, ’+, unbounded, 1) .
result Trace:
{’__[’$.Coin,’q.Coin,’q.Coin,’q.Coin],
’Marking,
rl ’M:Marking => ’__[’$.Coin,’M:Marking] [label(’add-$)] .}
{’__[’$.Coin,’$.Coin,’q.Coin,’q.Coin,’q.Coin],
’Marking,
rl ’$.Coin => ’c.Item [label(’buy-c)] .}
{’__[’$.Coin,’q.Coin,’q.Coin,’q.Coin,’c.Item],
’Marking,
rl ’$.Coin => ’__[’q.Coin,’a.Item] [label(’buy-a)] .}
{’__[’q.Coin,’q.Coin,’q.Coin,’q.Coin,’a.Item,’c.Item],
’Marking,
rl ’__[’q.Coin,’q.Coin,’q.Coin,’q.Coin] => ’$.Coin
[label(’change)] .}
{’__[’$.Coin,’a.Item,’c.Item],
’Marking,
rl ’$.Coin => ’c.Item [label(’buy-c)] .}
The operations metaSearchPath and metaSearch share caching, so calling one after the
other on the same arguments only performs a single search.
17.6.7 Rewriting using strategies: metaSrewrite
The operation metaSrewrite takes as arguments the metarepresentation of a module, the
metarepresentation of a term, the metapresentation of a strategy expression, a constant of sort
SrewriteOption, and a natural number. It rewrites the given term following the given strategy
in the given module.
op metaSrewrite :
Module Term Strategy SrewriteOption Nat ~> ResultPair? [special (...)] .
This function is the metarepresentation of the srewrite and dsrewrite commands, depending
on whether the SrewriteOption parameter is breadthFirst or depthFirst respectively. Like
in other similar descent functions, the last parameter allows enumerating the strategy solutions,
and evaluates to the constant failure when the given index is higher than the number of
solutions.
For example, in the QUEENS-STRAT module of Section 10.3, we can obtain all possible ways
of extending the partial solution 1 3 5, by enumerating the solutions of the expand strategy
defined in QUEENS-STRAT:
Maude> red in META-LEVEL : metaSrewrite([’QUEENS-STRAT], upTerm(1 3 5),
’expand[[empty]], breadthFirst, 0) .
result ResultPair:
{’__[’s_[’0.Zero],’s_^3[’0.Zero],’s_^5[’0.Zero],’s_^2[’0.Zero]],’NeList‘{Nat‘}}
Maude> red metaSrewrite([’QUEENS-STRAT], upTerm(1 3 5),
’expand[[empty]], breadthFirst, 1) .
result ResultPair:
{’__[’s_[’0.Zero],’s_^3[’0.Zero],’s_^5[’0.Zero],’s_^7[’0.Zero]],’NeList‘{Nat‘}}

<!-- manual page 457 -->
17.6. THE META-LEVEL MODULE: METALEVEL OPERATIONS 445
Maude> red metaSrewrite([’QUEENS-STRAT], upTerm(1 3 5),
’expand[[empty]], breadthFirst, 2) .
result ResultPair:
{’__[’s_[’0.Zero],’s_^3[’0.Zero],’s_^5[’0.Zero],’s_^8[’0.Zero]],’NeList‘{Nat‘}}
Maude> red metaSrewrite([’QUEENS-STRAT], upTerm(1 3 5),
’expand[[empty]], breadthFirst, 3) .
result ResultPair?:
failure
17.6.8 Unification: metaUnify,metaIrredundantUnify,metaDisjointUnify
and metaIrredundantDisjointUnify
The unification command of Section 13.4 is reflected in the META-LEVEL module by two descent
functions:
op metaUnify :
Module UnificationProblem Nat Nat ~> UnificationPair? [special (...)] .
op metaDisjointUnify :
Module UnificationProblem Nat Nat ~> UnificationTriple? [special (...)] .
These two metalevel functions work on unification problems constructed by means of the fol-
lowing signature:
sorts UnificandPair UnificationProblem .
subsort UnificandPair < UnificationProblem .
op _=?_ : Term Term -> UnificandPair [ctor prec 71] .
op _/\_ : UnificationProblem UnificationProblem -> UnificationProblem
[ctor assoc comm prec 73] .
The key difference between metaUnify and metaDisjointUnify is that the latter assumes
that the variables in the left and righthand unificands are to be considered disjoint even when
they are not so, and it generates each solution to the given unification problem not as a single
substitution, but as a pair of substitutions, one for left unificands and the other for right
unificands. This functionality is very useful for applications, such as critical-pair checking or
narrowing, where a disjoint copy of the terms or rules involved must always be computed before
unification is performed. Indeed, what the metaDisjointUnify operation avoids is precisely
the need for explicitly computing such disjoint copies. The need for two substitutions in each
solution is then obvious, since the terms in the given unification problem need not be made
explicitly disjoint, but their (accidentally) common variables must be treated differently, as if
they were disjoint.
Since it is convenient to reuse variable names from unifiers in new problems, for example
in narrowing, this is allowed via the third argument, which is the largest number n appearing
in a unificand variable of the form #n:Sort (see Section 13.4). The latest version of Maude
includes an alternative interface to variable reuse by using a Qid among ’#, ’% and ’@ instead
of a natural number in the third argument, indicating that it is the identifier ξused in unificand
variables of the form ξn:Sort.
op metaUnify :
Module UnificationProblem Qid Nat ~> UnificationPair? [special (...)] .
op metaDisjointUnify :
Module UnificationProblem Qid Nat ~> UnificationTriple? [special (...)] .

<!-- manual page 458 -->
446CHAPTER 17. REFLECTION, METALEVEL COMPUTATION, AND INTERNAL STRATEGIES
This avoids variable name clashes with the variable families ’#, ’% and ’@ used internally
by Maude for, respectively, unification modulo axioms in Chapter 13, variant unification in
Chapter 14, and narrowing in Chapter 15.
When we are interested in the minimal set of most general unifiers modulo axioms, we
should use the following two descent functions, which are defined only for the alternative of
using a Qid instead of a natural number in the third argument:
op metaIrredundantUnify :
Module UnificationProblem Qid Nat ~> UnificationPair? [special (...)] .
op metaIrredundantDisjointUnify :
Module UnificationProblem Qid Nat ~> UnificationTriple? [special (...)] .
As is usual for descent functions, the last argument in the function is used to select which
result is wanted, starting from 0. Caching is used so that if unifier i has just been returned,
requesting unifier i+ 1 gives rise to an incremental computation.
In the case where a call to metaUnify or metaDisjointUnify is invoked with a natural in
the third argument, the following constructors are used:
subsort UnificationPair < UnificationPair? .
subsort UnificationTriple < UnificationTriple? .
op {_,_} : Substitution Nat -> UnificationPair [ctor] .
op {_,_,_} : Substitution Substitution Nat -> UnificationTriple [ctor] .
as appropriate for the descent function. The final Nat component is the largest n occurring in
a fresh metavariable of the form #n:Sort. In this way, when we want to reuse variable names
from unifiers, the next invocation of the function can use this parameter to make sure that the
new variables generated are always fresh.
In the case where a call to metaUnify or metaDisjointUnify is invoked with a Qid in the
third argument, the following different constructors are used:
op {_,_} : Substitution Qid -> UnificationPair [ctor] .
op {_,_,_} : Substitution Substitution Qid -> UnificationTriple [ctor] .
and now they return the Qid specifying a variable family among ’#, ’% and ’@ instead of the
largest natural number used. Note that such Qid will indeed be different to the Qid given in
the third argument.
When no unifier with a given index exists, the constant
op noUnifier : -> UnificationPair? [ctor] .
or, respectively, the constant
op noUnifier : -> UnificationTriple? [ctor] .
are returned as appropriate for the corresponding descent function.
Recall that for unification modulo associative symbols no finite set of unifiers may exist, yet
a finite set is returned with a warning if the set may be incomplete (see Section 13.4.6). At the
metalevel, the role of this warning is played by the constant:
op noUnifierIncomplete : -> UnificationPair? [ctor] .
or, respectively, the constant
op noUnifierIncomplete : -> UnificationTriple? [ctor] .
which is returned when a finite set of most general unifiers cannot be ensured.
We can illustrate the use of these metalevel functions with a few examples. The first one
comes from Section 13.4, but moved up at the metalevel:

<!-- manual page 459 -->
17.6. THE META-LEVEL MODULE: METALEVEL OPERATIONS 447
Maude> reduce in META-LEVEL :
metaUnify(upModule(’UNIFICATION-EX1, false),
’f[’X:Nat, ’Y:NzNat] =? ’f[’Z:NzNat, ’U:Nat] /\
’V:NzNat =? ’f[’X:Nat, ’U:Nat], 0, 0) .
result UnificationPair:
{’U:Nat <- ’#1:NzNat ;
’V:NzNat <- ’f[’#2:NzNat, ’#1:NzNat] ;
’X:Nat <- ’#2:NzNat ;
’Y:NzNat <- ’#1:NzNat ;
’Z:NzNat <- ’#2:NzNat, 2}
If we replace the value 0 of the third argument by a Qid, e.g. ’#, we get the same output but
the second argument of the result will be either ’% or ’@.
The second example shows that we can request fresh variables with arbitrarily large num-
bering:
Maude> reduce in META-LEVEL :
metaUnify(upModule(’NAT, false),
’_+_[’X:Nat,’Y:Nat] =? ’_+_[’A:Nat,’B:Nat],
100000000000000000000, 0) .
result UnificationPair:
{’A:Nat <- ’_+_[’#100000000000000000001:Nat,
’#100000000000000000002:Nat] ;
’B:Nat <- ’_+_[’#100000000000000000003:Nat,
’#100000000000000000004:Nat] ;
’X:Nat <- ’_+_[’#100000000000000000001:Nat,
’#100000000000000000003:Nat] ;
’Y:Nat <- ’_+_[’#100000000000000000002:Nat,
’#100000000000000000004:Nat],
100000000000000000004}
The following example shows a similar unification problem but with much smaller number-
ings in fresh variables, and now involving an invocation of metaDisjointUnify.
Maude> reduce in META-LEVEL :
metaDisjointUnify(upModule(’NAT, false),
’_+_[’X:Nat, ’Y:Nat] =? ’_+_[’X:Nat, ’B:Nat], 0, 0) .
result UnificationTriple: {
’X:Nat <- ’_+_[’#1:Nat, ’#2:Nat] ;
’Y:Nat <- ’_+_[’#3:Nat, ’#4:Nat],
’B:Nat <- ’_+_[’#1:Nat, ’#3:Nat] ;
’X:Nat <- ’_+_[’#2:Nat, ’#4:Nat], 4}
Yet another example shows how using variable names in unification problems with larger
numbers than declared by the third argument generates a warning and no reduction.
Maude> reduce in META-LEVEL :
metaUnify(upModule(’NAT, false),
’_+_[’X:Nat,’Y:Nat] =? ’_+_[’#1:Nat,’Y:Nat], 0, 0) .
Warning: unsafe variable name #1:Nat in unification problem.
result [UnificationPair?]:

<!-- manual page 460 -->
448CHAPTER 17. REFLECTION, METALEVEL COMPUTATION, AND INTERNAL STRATEGIES
metaUnify(th ’NAT is
including ’NAT .
sorts none .
none
none
none
none
none
endth,
’_+_[’X:Nat, ’Y:Nat] =? ’_+_[’#1:Nat, ’Y:Nat], 0, 0)
Similarly, the following example shows how using a variable of the form ξn:Sort for ξbeing
either ’#, ’% or ’@ but different from the Qid among ’#, ’% and ’@ given in the third argument
generates a warning and no reduction.
Maude> reduce in META-LEVEL :
metaUnify(upModule(’NAT, false),
’_+_[’X:Nat,’Y:Nat] =? ’_+_[’#1:Nat,’Y:Nat], ’%, 0) .
Warning: unsafe variable name #1:Nat in unification problem.
result [UnificationPair?]:
metaUnify(th ’NAT is
including ’NAT .
sorts none .
none
none
none
none
none
endth,
’_+_[’X:Nat, ’Y:Nat] =? ’_+_[’#1:Nat, ’Y:Nat], ’%, 0)
Note that if the variables are of the form ξn:Sort for ξ being neither ’#, ’% nor ’@, no warning
is given, since there will be no conflict with the internal use of variable families ’#, ’% and ’@.
And finally an example of incomplete unification for the associative case. If we move to
the metalevel the unification problem with an infinite set of most general unifiers, e.g., the
unification problem 0:X=? X:0 of Section 13.4.6, we get the first unifier of the family:
Maude> reduce in META-LEVEL :
metaUnify(upModule(’UNIFICATION-EX4, true),
’_:_[’0.Nat,’X:NList] =? ’_:_[’X:NList,’0.Nat], 0, 0) .
Warning: Unification modulo the theory of operator _:_ has encountered
an instance for which it may not be complete.
result UnificationPair: {
’X:NList <- ’0.Zero,0}
but successive calls for unifiers get the constant noUnifierIncomplete:
Maude> reduce in META-LEVEL :
metaUnify(upModule(’UNIFICATION-EX4, true),
’_:_[’0.Nat,’X:NList] =? ’_:_[’X:NList,’0.Nat], 0, 1) .
result UnificationPair?: (noUnifierIncomplete).UnificationPair?

<!-- manual page 461 -->
17.6. THE META-LEVEL MODULE: METALEVEL OPERATIONS 449
Note that we got the constant noUnifierIncompleteinstead of the noUnifierconstant, which
is the output for the case of a finitary set of most general unifiers.
Several auxiliary functions have been defined by equations, allowing easy extraction of
information.
op getSubstitution : UnificationPair -> Substitution .
op getVariableFamily : UnificationPair -> Qid .
op getLhsSubstitution : UnificationTriple -> Substitution .
op getRhsSubstitution : UnificationTriple -> Substitution .
op getVariableFamily : UnificationTriple -> Qid .
Since it is quite common to apply a substitution to a term, we have included such feature
as a function defined by equations
op applySubstitution : Module Term Substitution -> Term .
17.6.9 Variants: metaGetVariant
The procedure for variant generation of Section 14.4 is also available at the metalevel of Maude
thanks to the metaGetVariant and metaGetIrredundantVariant functions provided in the
META-LEVEL module.
op metaGetVariant : Module Term TermList Nat Nat ~> Variant?
[special (...)] .
op metaGetIrredundantVariant : Module Term TermList Nat Nat ~> Variant?
[special (...)] .
The third argument allows a list of irreducible terms, see Section 14.5 for details. As in Sec-
tion 17.6.8, it is convenient to reuse variable names from terms; this is allowed via the fourth
argument, which is the largest number n appearing in fresh variables of the form #n:Sort or
%n:Sort. The latest version of Maude includes an alternative interface to these two functions
by using a Qid among ’#, ’% and ’@ instead of a natural number in the fourth argument,
indicating that it is the identifier ξ used in unificand variables of the form ξn:Sort.
op metaGetVariant : Module Term TermList Qid Nat ~> Variant?
[special (...)] .
op metaGetIrredundantVariant : Module Term TermList Qid Nat ~> Variant?
[special (...)] .
This avoids variable name clashes with the variable families ’#, ’% and ’@ used internally
by Maude for, respectively, unification modulo axioms in Chapter 13, variant unification in
Chapter 14, and narrowing in Chapter 15.
As usual for descent functions, the last argument in the function is used to select which
result is wanted, starting from 0. Caching is used so that if variant i has just been returned,
requesting unifier i+ 1 gives rise to an incremental computation.
In the case where a call to metaGetVariant or metaGetIrredundantVariant is invoked
with a natural in the fourth argument the following constructors are used:
sorts Variant Variant? .
subsort Variant < Variant? .
op {_,_,_,_,_} : Term Substitution Nat Parent Bool -> Variant [ctor] .
op noVariant : -> Variant? [ctor] .
op noVariantIncomplete : -> Variant? [ctor] .

<!-- manual page 462 -->
450CHAPTER 17. REFLECTION, METALEVEL COMPUTATION, AND INTERNAL STRATEGIES
Again, the third argument denotes the largest number nused in the fresh variables appearing in
the solutions. The fourth and fifth arguments are useful for applications based on the execution
narrowing tree rather than the set of variants, see the example below.
In the case where a call to metaGetVariant or metaGetIrredundantVariant is invoked
with a Qid among ’#, ’% and ’@ in the fourth argument, the following different constructor is
used:
op {_,_,_,_,_} : Term Substitution Qid Parent Bool -> Variant [ctor] .
and now it returns the Qid specifying a variable family instead of the largest natural number
used. Note that such Qid will indeed be different to the Qid given in the fourth argument.
We can illustrate the use of this metalevel function with the first variant of the configuration
< $ q q X:Marking Y:Marking >.
Maude> reduce in META-LEVEL :
metaGetVariant(upModule(’VARIANT-VENDING-MACHINE, true),
’<_>[’__[’$.Coin,’q.Coin,’q.Coin,’X:Marking,’Y:Marking]], empty, 0, 0) .
result Variant: {’<_>[’__[’$.Coin,’q.Coin,’q.Coin,’#1:Marking,’#2:Marking]],
’X:Marking <- ’#1:Marking ;
’Y:Marking <- ’#2:Marking,2,none,false}
Then the second possible variant:
Maude> reduce in META-LEVEL :
metaGetVariant(upModule(’VARIANT-VENDING-MACHINE, true),
’<_>[’__[’$.Coin,’q.Coin,’q.Coin,’X:Marking,’Y:Marking]], empty, 0, 1) .
result Variant: {’<_>[’__[’$.Coin,’$.Coin,’%1:Marking,’%2:Marking]],
’X:Marking <- ’__[’q.Coin,’q.Coin,’%1:Marking] ;
’Y:Marking <- ’%2:Marking,2,0,true}
Then the third possible variant:
Maude> reduce in META-LEVEL :
metaGetVariant(upModule(’VARIANT-VENDING-MACHINE, true),
’<_>[’__[’$.Coin,’q.Coin,’q.Coin,’X:Marking,’Y:Marking]], empty, 0, 2) .
result Variant: {’<_>[’__[’$.Coin,’$.Coin,’%1:Marking,’%2:Marking]],
’X:Marking <- ’__[’q.Coin,’%1:Marking] ;
’Y:Marking <- ’__[’q.Coin,’%2:Marking],2,0,true}
Then the fourth possible variant:
Maude> reduce in META-LEVEL :
metaGetVariant(upModule(’VARIANT-VENDING-MACHINE, true),
’<_>[’__[’$.Coin,’q.Coin,’q.Coin,’X:Marking,’Y:Marking]], empty, 0, 3) .
result Variant: {’<_>[’__[’$.Coin,’$.Coin,’%1:Marking,’%2:Marking]],
’X:Marking <- ’%1:Marking ;
’Y:Marking <- ’__[’q.Coin,’q.Coin,’%2:Marking],2,0,false}
Then the fifth possible variant:
Maude> reduce in META-LEVEL :
metaGetVariant(upModule(’VARIANT-VENDING-MACHINE, true),
’<_>[’__[’$.Coin,’q.Coin,’q.Coin,’X:Marking,’Y:Marking]], empty, 0, 4) .
result Variant: {’<_>[’__[’$.Coin,’$.Coin,’$.Coin,’#1:Marking,’#2:Marking]],
’X:Marking <- ’__[’q.Coin,’q.Coin,’q.Coin,’#1:Marking] ;
’Y:Marking <- ’__[’q.Coin,’q.Coin,’q.Coin,’#2:Marking],2,1,false}
And there are no more variants.

<!-- manual page 463 -->
17.6. THE META-LEVEL MODULE: METALEVEL OPERATIONS 451
< $ q q X Y >
{X →q q X1,
Y →Y1}
0
{X →q X2,
Y →q Y2}
{X →X3,
Y →q q Y3}
< $ $ X1 Y1 >
< $ $ X2 Y2 >
< $ $ X3 Y3 >
1 2 3
{X1 →q X4,
Y1 →q q q X4}
< $ $ $ X4 Y4 >
4
Figure 17.2: Folding variant narrowing tree for the term < $ q q X Y >.
Maude> reduce in META-LEVEL :
metaGetVariant(upModule(’VARIANT-VENDING-MACHINE, true),
’<_>[’__[’$.Coin,’q.Coin,’q.Coin,’X:Marking,’Y:Marking]], empty, 0, 5) .
result Variant?: noVariant
Using the fourth and fifth arguments of each returned variant, we can reconstruct the
execution narrowing tree of Figure 17.2. The fourth argument of each variant is the identifier
of the parent variant; the identifier of each variant is indeed the last argument of its associated
call to metaGetVariant. The fifth argument is a Boolean: true meaning that there is at least
one other variant in that level of the narrowing tree, and false meaning that this is the last one
in that level of the narrowing tree. Note that variants return the whole composed substitution
and the intermediate unifier shown in Figure 17.2 between variants 1 and 4 has to be extracted
manually.
We can reproduce the example of the vending machine using irreducible terms showed in
Section 14.5 as follows.
Maude> reduce in META-LEVEL :
metaGetVariant(upModule(’VARIANT-VENDING-MACHINE, true),
’<_>[’__[’$.Coin,’q.Coin,’q.Coin,’X:Marking,’Y:Marking]],
(’__[’q.Coin,’q.Coin,’X:Marking],’__[’q.Coin.’X:Marking], ’X:Marking),
0, 0) .
result Variant: {’<_>[’__[’$.Coin,’q.Coin,’q.Coin,’#1:Marking,’#2:Marking]],
’X:Marking <- ’#1:Marking ;
’Y:Marking <- ’#2:Marking,2,none,false}
The command prints only three of the five variants returned by the previous call without any ir-
reducibleterm. ThevariantswhereX:Markingismappedtoq q %1:Markingorq q q %1:Marking
are discarded, since they violate the condition that q q X:Marking must be irreducible under
substitution.
Maude> reduce in META-LEVEL :
metaGetVariant(upModule(’VARIANT-VENDING-MACHINE, true),
’<_>[’__[’$.Coin,’q.Coin,’q.Coin,’X:Marking,’Y:Marking]],
(’__[’q.Coin,’q.Coin,’X:Marking],’__[’q.Coin.’X:Marking], ’X:Marking),
0, 3) .
result Variant?: noVariant

<!-- manual page 464 -->
452CHAPTER 17. REFLECTION, METALEVEL COMPUTATION, AND INTERNAL STRATEGIES
Letusalsoshowanexampleofanincompletevariantgenerationatthemetalevel. Ifwemove
tothemetaleveltheincompletevariantgenerationfortermduplicate(prefix(L) : tail(L))
of Section 14.7, we get the first variant:
Maude> reduce in META-LEVEL :
metaGetVariant(upModule(’VARIANT-UNIFICATION-ASSOC, true),
’duplicate[’_:_[’prefix[’L:NList],’tail[’L:NList]]], empty, 0, 0) .
result Variant: {’duplicate[’_:_[’prefix[’#1:NList],’tail[’#1:NList]]],
’L:NList <- ’#1:NList,1,none,false}
And when we ask for the eighth variant, we get the constant noVariantIncomplete:
Maude> reduce in META-LEVEL :
metaGetVariant(upModule(’VARIANT-UNIFICATION-ASSOC, true),
’duplicate[’_:_[’prefix[’L:NList],’tail[’L:NList]]], empty, 0, 7) .
result Variant?: noVariantIncomplete
Note that we got the constant noVariantIncompleteinstead of the noVariantconstant, which
is the output for the case of a finitary set of variants.
17.6.10 Variant Matching and Unification: metaVariantUnify,
metaVariantDisjointUnify, and metaVariantMatch
The procedure for variant-based equational unification of Section 14.9 is also available at the
metalevel by means of the following functions provided in the META-LEVEL module.
op metaVariantUnify :
Module UnificationProblem TermList Nat Nat ~> UnificationPair?
[special (...)] .
op metaVariantDisjointUnify :
Module UnificationProblem TermList Nat Nat ~> UnificationTriple?
[special (...)] .
The unification problems and the result sort are the same as in Section 17.6.8. The third
argument allows a list of irreducible terms, see Section 14.10 for details.
sorts UnificandPair UnificationProblem .
subsort UnificandPair < UnificationProblem .
op _=?_ : Term Term -> UnificandPair [ctor prec 71] .
op _/\_ : UnificationProblem UnificationProblem -> UnificationProblem
[ctor assoc comm prec 73] .
subsort UnificationPair < UnificationPair? .
subsort UnificationTriple < UnificationTriple? .
op {_,_} : Substitution Nat -> UnificationPair [ctor] .
op {_,_,_} : Substitution Substitution Nat -> UnificationTriple [ctor] .
The latest version of Maude includes an alternative interface to these two functions using
a Qid instead of a natural number in the fourth argument, which is the identifier ξ used in
unificand variables of the form ξn:Sort.
op metaVariantUnify :
Module UnificationProblem TermList Qid VariantOptionSet Nat ~> UnificationPair?
[special (...)] .
op metaVariantDisjointUnify :

<!-- manual page 465 -->
17.6. THE META-LEVEL MODULE: METALEVEL OPERATIONS 453
Module UnificationProblem TermList Qid VariantOptionSet Nat ~> UnificationTriple?
[special (...)] .
In the case where a call to metaVariantUnify or metaVariantDisjointUnify is invoked with
a Qid in the fourth argument, the following different constructors are used:
op {_,_} : Substitution Qid -> UnificationPair [ctor] .
op {_,_,_} : Substitution Substitution Qid -> UnificationTriple [ctor] .
and now they return the Qid specifying a variable family instead of the largest natural number
used. Note that such Qid will indeed be different to the Qid given in the third argument.
When we are interested in the minimal set of most general unifiers, there is no alternative
command, as in Section 17.6.8, and both options filter and delay must be used in the fifth
argument of the commands:
sorts VariantOption VariantOptionSet .
subsort VariantOption < VariantOptionSet .
ops delay filter : -> VariantOption [ctor] .
op none : -> VariantOptionSet [ctor] .
op __ : VariantOptionSet VariantOptionSet -> VariantOptionSet
[ctor assoc comm id: none] .
The filter flag indicates that filtering is activated but it can be done either during equational
unification (the delay flag is not included) or after all equational unifiers have been computed
(the delay flag is included); the latter being the method used in the filtered variant unify
command of Section 14.9. Note that the version of the command with a counter for new
variables does not include this fifth argument for options and, thus, cannot return the minimal
set of unifiers.
We can illustrate the use of this metalevel function with the variant unification of the two
terms of Section 14.9: < q q X:Marking > and < $ Y:Marking >:
Maude> reduce in META-LEVEL :
metaVariantUnify(upModule(’VARIANT-VENDING-MACHINE, true),
’<_>[’__[’q.Coin,’q.Coin,’X:Marking]] =?
’<_>[’__[’$.Coin,’Y:Marking]],
empty, ’@, none, 0) .
result UnificationPair: {
’X:Marking <- ’__[’$.Coin,’%1:Marking] ;
’Y:Marking <- ’__[’q.Coin,’q.Coin,’%1:Marking],’%}
Maude> reduce in META-LEVEL :
metaVariantUnify(upModule(’VARIANT-VENDING-MACHINE, true),
’<_>[’__[’q.Coin,’q.Coin,’X:Marking]] =?
’<_>[’__[’$.Coin,’Y:Marking]],
empty, ’@, none, 1) .
result UnificationPair: {
’X:Marking <- ’__[’q.Coin,’q.Coin,’#1:Marking] ;
’Y:Marking <- ’#1:Marking,’#}
Maude> reduce in META-LEVEL :
metaVariantUnify(upModule(’VARIANT-VENDING-MACHINE, true),
’<_>[’__[’q.Coin,’q.Coin,’X:Marking]] =?
’<_>[’__[’$.Coin,’Y:Marking]],
empty, ’@, none, 2) .
result UnificationPair?: (noUnifier).UnificationPair?

<!-- manual page 466 -->
454CHAPTER 17. REFLECTION, METALEVEL COMPUTATION, AND INTERNAL STRATEGIES
And if you include the flags for the minimal set of most general unifiers, it returns only one.
Maude> reduce in META-LEVEL :
metaVariantUnify(upModule(’VARIANT-VENDING-MACHINE, true),
’<_>[’__[’q.Coin,’q.Coin,’X:Marking]] =?
’<_>[’__[’$.Coin,’Y:Marking]],
empty, ’@, filter delay, 0) .
result UnificationPair: {
’X:Marking <- ’__[’q.Coin,’q.Coin,’#1:Marking] ;
’Y:Marking <- ’#1:Marking,’#}
Maude> reduce in META-LEVEL :
metaVariantUnify(upModule(’VARIANT-VENDING-MACHINE, true),
’<_>[’__[’q.Coin,’q.Coin,’X:Marking]] =?
’<_>[’__[’$.Coin,’Y:Marking]],
empty, ’@, filter delay, 1) .
result UnificationPair?: (noUnifier).UnificationPair?
Let us also illustrate the use of incomplete variant unification by moving to the metalevel
the incomplete unification problem of Section 14.12: head(L:NList) =? last(L:NList) /\
prefix(L:NList) =? tail(L:NList).
Maude> reduce in META-LEVEL :
metaVariantUnify(upModule(’VARIANT-UNIFICATION-ASSOC, true),
’head[’L:NList] =? ’last[’L:NList] /\
’prefix[’L:NList] =? ’tail[’L:NList], empty, 0, 0) .
Warning: Unification modulo the theory of operator _:_ has encountered
an instance for which it may not be complete.
result UnificationPair: {
’L:NList <- ’_:_[’%1:Nat,’%1:Nat,’%1:Nat],1}
And when we try to obtain the third unifier we get the constant noUnifierIncomplete.
Maude> reduce in META-LEVEL :
metaVariantUnify(upModule(’VARIANT-UNIFICATION-ASSOC, true),
’head[’L:NList] =? ’last[’L:NList] /\
’prefix[’L:NList] =? ’tail[’L:NList], empty, 0, 2) .
result UnificationPair?: (noUnifierIncomplete).UnificationPair?
Several auxiliary functions have been defined by equations, allowing easy extraction of
information.
op getTerm : Variant -> Term .
op getSubstitution : Variant -> Substitution .
op getVariableFamily : Variant -> Qid .
op getParent : Variant -> Parent .
op getMoreVariantsInLayerFlag : Variant -> Bool .
The procedure for variant-based equational matching of Section 14.13 is also available at
the metalevel by means of the following function provided in the META-LEVEL module.
op metaVariantMatch :
Module MatchingProblem TermList Qid VariantOptionSet Nat ~> Substitution?
The new matching problems are as follows.

<!-- manual page 467 -->
17.6. THE META-LEVEL MODULE: METALEVEL OPERATIONS 455
sorts PatternSubjectPair MatchingProblem .
subsort PatternSubjectPair < MatchingProblem .
op _<=?_ : Term Term -> PatternSubjectPair [ctor prec 71] .
op _/\_ : MatchingProblem MatchingProblem -> MatchingProblem [ctor assoc comm prec 73] .
We can illustrate the use of this metalevel function with the variant matching of the two
terms of Section 14.13: < q q X:Marking > and < $ Y:Marking >:
Maude> reduce in META-LEVEL :
metaVariantMatch(upModule(’VARIANT-VENDING-MACHINE, true),
’<_>[’__[’q.Coin,’q.Coin,’X:Marking]] <=?
’<_>[’__[’$.Coin,’Y:Marking]],
empty, ’@, none, 0) .
result Assignment:
’X:Marking <- ’__[’q.Coin,’q.Coin,’Y:Marking]
Note that the delay and filter constants of the sort VariantOptionSet have no effect for
variant matching and we should always use none.
17.6.11 Narrowing: metaNarrowingApply, metaNarrowingSearch and
metaNarrowingSearchPath
Narrowing is also available at the metalevel by using the functions metaNarrowingApply,
metaNarrowingSearch and metaNarrowingSearchPath. Note that there is no user level com-
mand associated to the metalevel function metaNarrowingApply.
The invocation of just one narrowing step is reproduced by function metaNarrowingApply:
op metaNarrowingApply :
Module Term TermList Qid VariantOptionSet Nat -> NarrowingApplyResult?
[special ...] .
The result sort is defined by means of the following data:
sorts NarrowingApplyResult NarrowingApplyResult? .
subsort NarrowingApplyResult < NarrowingApplyResult? .
op {_,_,_,_,_,_,_} : Term Type Context Qid Substitution Substitution Qid
-> NarrowingApplyResult
[ctor format (d n++i d d d ni d ni d d d d d ni n--i d)] .
op failure : -> NarrowingApplyResult? [ctor] .
op failureIncomplete : -> NarrowingApplyResult? [ctor] .
The third argument allows a list of irreducible terms, as in Sections 17.6.9 and 17.6.10. The
fourth argument provides the identifier ξ used in variables of the form ξn:Sort appearing in
the given term, again as in Sections 17.6.9 and 17.6.10. The fifth argument provides the options
for variant unification, again as in Sections 17.6.9 and 17.6.10. The last argument is the chosen
narrowing step. If there is no solution, the failure constant is returned if no incompleteness
situation related to associative unification has been found; otherwise, the failureIncomplete
constant is returned.
For the NARROWING-VENDING-MACHINE system module introduced at the beginning of Sec-
tion 15.6, the following one-step narrowing command can be given
Maude> reduce in META-LEVEL :
metaNarrowingApply(upModule(’NARROWING-VENDING-MACHINE, false),
’<_>[’M:Money], empty, ’@, none, 0) .
result NarrowingApplyResult: {

<!-- manual page 468 -->
456CHAPTER 17. REFLECTION, METALEVEL COMPUTATION, AND INTERNAL STRATEGIES
’<_>[’__[’a.Item,’q.Coin,’%1:Money]],’State,
[],
’buy-a,
’M:Money <- ’__[’$.Coin,’%1:Money],
’M:Marking <- ’%1:Money,
’%
}
Note that two substitutions are returned, one for the input term and another for the lefthand
side of the applied rule.
The narrowing-based reachability analysis of Section 15.6 is available at the metalevel by
using the function metaNarrowingSearch:
op metaNarrowingSearch :
Module Term Term Qid Bound Qid VariantOptionSet Nat -> NarrowingSearchResult?
[special ...] .
The result sort is defined by means of the following data:
sorts NarrowingSearchResult NarrowingSearchResult? .
subsort NarrowingSearchResult < NarrowingSearchResult? .
op {_,_,_,_,_,_} : Term Type Substitution Qid Substitution Qid
-> NarrowingSearchResult
[ctor format (d n++i d d d d d ni d d d ni n--i d)] .
op failure : -> NarrowingSearchResult? [ctor] .
op failureIncomplete : -> NarrowingSearchResult? [ctor] .
ThefirstQidargumentmetarepresentstheappropriatesearcharrow,similartothemetaSearch
command (see Section 17.6.6). The second Qid determines whether folding is applied or not,
see Section 15.6.3. The constant ’none indicates that standard narrowing without any folding
is applied, as the vu-narrow command of Section 15.6. The constant ’match indicates that
folding narrowing is applied, as the fvu-narrow command of Section 15.6.3. The seventh ar-
gument provides the options for variant unification, again as in Sections 17.6.9 and 17.6.10.
For the bounds, the Bound one is the maximum length of the narrowing sequences, whereas
the Nat is the chosen solution (in order to provide all solutions in a sequential way, as many
metalevel commands in Maude do). If there is no solution, the failure constant is returned
if no incompleteness situation related to associative unification has been found; otherwise, the
failureIncomplete constant is returned.
For the NARROWING-VENDING-MACHINE system module introduced at the beginning of Sec-
tion 15.6, the following search command considered above
Maude> vu-narrow [1] in NARROWING-VENDING-MACHINE : < M:Money > =>* < a c > .
can be specified at the metalevel as follows, where ’<_>[’M:Money]is the metarepresentation of
the state < M:Money >, ’<_>[’__[’a.Item,’c.Item]] is the metarepresentation of the state
< a c >, and we use the coherence completion of the NARROWING-VENDING-MACHINE module
given above.
Maude> reduce in META-LEVEL :
metaNarrowingSearch(
upModule(’NARROWING-VENDING-MACHINE, false),
’<_>[’M:Money], ’<_>[’__[’a.Item,’c.Item]], ’*, unbounded, ’match, none, 0) .
result NarrowingSearchResult: {
’<_>[’__[’a.Item,’c.Item,’#1:Money]],’State,
’M:Money <- ’__[’$.Coin,’q.Coin,’q.Coin,’q.Coin,’#1:Money],

<!-- manual page 469 -->
17.6. THE META-LEVEL MODULE: METALEVEL OPERATIONS 457
’#,
’@
’#1:Money <- ’empty.Money,
}
Note that we obtain the very same solution, where the output contains the actual output term,
its type, the accumulated substitution, the identifier used for creating fresh variables (# in this
example), and the variant-unifier.
Moreover, we can also obtain the narrowing sequence associated to a narrowing-based reach-
ability command with the function metaNarrowingSearchPath:
op metaNarrowingSearchPath :
Module Term Term Qid Bound Qid VariantOptionSet Nat -> NarrowingSearchPathResult?
[special ...] .
The result sort is defined by means of the following data:
sorts NarrowingSearchPathResult NarrowingSearchPathResult? .
subsort NarrowingStep < NarrowingTrace .
subsort NarrowingSearchPathResult < NarrowingSearchPathResult? .
op {_,_,_,_,_,_,_} : Context Qid Substitution Qid Term Type Substitution
-> NarrowingStep
[ctor format (ni n++i d ni d d d ni d ni d d d d n--i d)] .
op nil : -> NarrowingTrace [ctor] .
op __ : NarrowingTrace NarrowingTrace -> NarrowingTrace [ctor assoc id: nil] .
op {_,_,_,_,_,_} : Term Type Substitution NarrowingTrace Substitution Qid
-> NarrowingSearchPathResult
[ctor format (d n++i d d d d d d d d d ni n--i d)] .
op failure : -> NarrowingSearchPathResult? [ctor] .
op failureIncomplete : -> NarrowingSearchPathResult? [ctor] .
It works in exactly the same way as metaNarrowingSearch but providing as a result a more
detailed data structure. If we redo the previous metaNarrowingSearch computation but using
this time the metaNarrowingSearchPath function, we obtain:
Maude> reduce in META-LEVEL :
metaNarrowingSearchPath(
upModule(’NARROWING-VENDING-MACHINE, false),
’<_>[’M:Money], ’<_>[’__[’a.Item,’c.Item]], ’*, unbounded, ’none, none, 0) .
result NarrowingSearchPathResult: {
’<_>[’#1:Money],’State,
’M:Money <- ’#1:Money,
{
[],
’buy-a,
’#1:Money <- ’__[’$.Coin,’@1:Money] ;
’M:Marking <- ’@1:Money,
’@,
’<_>[’__[’a.Item,’q.Coin,’@1:Money]],’State,
’M:Money <- ’__[’$.Coin,’@1:Money]
}
{
[],
’buy-c,
’@1:Money <- ’__[’q.Coin,’q.Coin,’q.Coin,’#1:Money] ;

<!-- manual page 470 -->
458CHAPTER 17. REFLECTION, METALEVEL COMPUTATION, AND INTERNAL STRATEGIES
},
’@
’M:Marking <- ’__[’a.Item,’#1:Money],
’#,
’<_>[’__[’a.Item,’c.Item,’#1:Money]],’State,
’M:Money <- ’__[’$.Coin,’q.Coin,’q.Coin,’q.Coin,’#1:Money]
’#1:Money <- ’empty.Money,
}
The data structure NarrowingStep, which is the basic element of NarrowingTrace, is very
similar to the data structure ResultTriple but it contains a sequence of narrowing results
instead of only the final result, each one together with the rule that has been used in that
narrowing step.
Several auxiliary functions have been defined by equations, allowing easy extraction of
information.
op getTerm : NarrowingApplyResult -> Term .
op getType : NarrowingApplyResult -> Type .
op getContext : NarrowingApplyResult -> Context .
op getLabel : NarrowingApplyResult -> Qid .
op getTermSubstitution : NarrowingApplyResult -> Substitution .
op getRuleSubstitution : NarrowingApplyResult -> Substitution .
op getVariableFamily : NarrowingApplyResult -> Qid .
op getTerm : NarrowingSearchResult -> Term .
op getType : NarrowingSearchResult -> Type .
op getAccumulatedSubstitution : NarrowingSearchResult -> Substitution .
op getStateVariableFamily : NarrowingSearchResult -> Qid .
op getUnifier : NarrowingSearchResult -> Substitution .
op getUnifierVariableFamily : NarrowingSearchResult -> Qid .
op getInitialTerm : NarrowingSearchPathResult -> Term .
op getInitialType : NarrowingSearchPathResult -> Type .
op getInitialSubstitution : NarrowingSearchPathResult -> Substitution .
op getTrace : NarrowingSearchPathResult -> NarrowingTrace .
op getUnifier : NarrowingSearchPathResult -> Substitution .
op getUnifierVariableFamily : NarrowingSearchPathResult -> Qid .
op getContext : NarrowingStep -> Context .
op getLabel : NarrowingStep -> Qid .
op getUnifier : NarrowingStep -> Substitution .
op getUnifierVariableFamily : NarrowingStep -> Qid .
op getTerm : NarrowingStep -> Term .
op getType : NarrowingStep -> Type .
op getAccumulatedSubstitution : NarrowingStep -> Substitution .

<!-- manual page 471 -->
17.6. THE META-LEVEL MODULE: METALEVEL OPERATIONS 459
17.6.12 Checking satisfiability modulo theories: metaCheck
The SMT check command of Chapter 16 is also available at the metalevel by using the function
metaCheck:
op metaCheck : Module Term ~> Bool [special (...)] .
The reflection of the SMT signatures follows the normal Maude metalevel conventions, e.g.,
2 becomes ’2.Integer and 1/3 becomes ’1/3.Real.
Consider the following example.
fmod META-CHECK is
pr META-LEVEL .
pr REAL-INTEGER .
vars W X Y Z : Boolean .
vars I J K L : Integer .
vars P Q R S : Real .
endfm
Maude> red in META-LEVEL :
metaCheck([’META-CHECK],
upTerm( (I > J ? I : J) === I or (I > J ? I : J) === J )) .
rewrites: 3 in 6ms cpu (6ms real) (456 rewrites/second)
result Bool: (true).Bool
Maude> red in META-LEVEL :
metaCheck([’META-CHECK],
upTerm( not((I > J ? I : J) === I or (I > J ? I : J) === J ) )) .
rewrites: 3 in 0ms cpu (0ms real) (13824 rewrites/second)
result Bool: (false).Bool
Maude> red in META-LEVEL :
metaCheck([’META-CHECK],
upTerm( (I > J ? I : J) =/== I and (I > J ? I : J) =/== J )) .
rewrites: 3 in 0ms cpu (0ms real) (13698 rewrites/second)
result Bool: (false).Bool
Maude> red in META-LEVEL :
metaCheck([’META-CHECK],
upTerm( (I > J ? I : J) =/== I or (I > J ? I : J) =/== J )) .
rewrites: 3 in 0ms cpu (0ms real) (12931 rewrites/second)
result Bool: (true).Bool
Here metaCheck returns true if the SMT solver responds with sat and false otherwise;
it can occasionally produce results other than sat and unsat, for example if it cannot decide
satisfiability and Maude’s response may change in the future.
Examples of the use of metaCheck in the analysis of time-aware security protocols to reduce
infinite search to a finite one can be found in [103, 104].
17.6.13 Parsing and pretty-printing
metaParse
The (partial) operation metaParse takes as arguments the metarepresentation of a module, a
set of metarepresented variables that are declared in the context where the term is parsed, a

<!-- manual page 472 -->
460CHAPTER 17. REFLECTION, METALEVEL COMPUTATION, AND INTERNAL STRATEGIES
list of quoted identifiers metarepresenting a list of tokens, and a value of the sort Type?, i.e.,
either the metarepresentation of a component or the constant anyType.
sort Type? .
subsort Type < Type? .
op anyType : -> Type? [ctor] .
sorts NeVariableSet VariableSet .
subsort Variable < NeVariableSet < VariableSet < QidSet .
subsort NeVariableSet < NeQidSet .
subsort EmptyQidSet < VariableSet .
op _;_ : VariableSet VariableSet -> VariableSet [ctor ditto] .
op _;_ : NeVariableSet VariableSet -> NeVariableSet [ctor ditto] .
sort ResultPair? .
subsort ResultPair < ResultPair? .
op noParse : Nat -> ResultPair? [ctor] .
op ambiguity : ResultPair ResultPair -> ResultPair? [ctor] .
op metaParse : Module VariableSet QidList Type? ~> ResultPair? [special (...)] .
The operation metaParse reflects the parse command in Maude (see Section 3.9.4); that
is, it tries to parse the given list of tokens as a term of the given type in the module given as
first argument; the constant anyType allows any component. If metaParse succeeds, it returns
the metarepresentation of the parsed term with its corresponding sort or kind. Otherwise, it
returns:
• noParse(n) if there was no parse, where n is the index of the first bad token (counting
from 0), or the number of tokens in the case of unexpected end of input; or
• ambiguity(r1, r2) if there were multiple parses, where r1 and r2 are the result pairs
corresponding to two distinct parses.
These are simple examples of using metaParse:
Maude> reduce in META-LEVEL :
metaParse(upModule(’VENDING-MACHINE, false), none, ’$ ’q ’q ’q, ’Marking) .
result ResultPair:
{’__[’$.Coin,’__[’q.Coin,’__[’q.Coin,’q.Coin]]],’Marking}
Maude> reduce in META-LEVEL :
result ResultPair?: noParse(2)
metaParse(upModule(’VENDING-MACHINE, false), none, ’$ ’q ’d ’q, ’Marking) .
Attheobjectlevel, variablesdeclaredinamodulecanbewrittenintermswithoutanexplicit
type annotation, like M instead of M:Marking. Although the metarepresentation of a module
does not include variable declarations, a set of sort VariableSet containing the metarepresen-
tations of the declared variables can be specified in the second argument of metaParse:
Maude> reduce in META-LEVEL :
metaParse(upModule(’VENDING-MACHINE, false), ’M1:Marking,
’$ ’M1 ’M2:Marking, ’Marking) .
result ResultPair: {’__[’$.Coin,’__[’M1:Marking,’M2:Marking]],’Marking}
For backwards compatibility, a variation of metaParse providing an empty set of variables is
available:
op metaParse : Module QidList Type? ~> ResultPair? .
eq metaParse(M:Module, Q:QidList, T:Type?)
= metaParse(M:Module, none, Q:QidList, T:Type?) .

<!-- manual page 473 -->
17.6. THE META-LEVEL MODULE: METALEVEL OPERATIONS 461
metaPrettyPrint
The (partial) operation metaPrettyPrint takes as arguments the metarepresentations of a
module R(first argument) and of a term t (third argument) together with a set of declared
variables (second argument), printing options (fourth argument), and a set of operator names
whose arguments should be concealed (fifth argument). It returns a list of quoted identifiers
that metarepresents the string of tokens produced by pretty-printing the term tin the signature
of R.
op metaPrettyPrint : Module VariableSet Term PrintOptionSet QidSet ~> QidList
[special (...)] .
Pretty-printing a term involves more than just naively using the mixfix syntax for operators.
Precedence and gathering information and the relative positions of underscores in an operator
and its parent in the term must be considered to determine whether parentheses need to be
inserted around any given subterm to avoid ambiguity. If there is ad-hoc overloading in the
module, additional checks must be done to determine if and where sort disambiguation syntax
is needed. Moreover, any variable that has been passed to the second argument of the function
must be printed without its type annotation, as terms are printed in an object-level module.
The print options argument is built with the following syntax:
sorts PrintOption PrintOptionSet .
subsort PrintOption < PrintOptionSet .
ops mixfix with-parens flat format number rat with-sorts : -> PrintOption [ctor] .
op none : -> PrintOptionSet [ctor] .
op __ : PrintOptionSet PrintOptionSet -> PrintOptionSet [ctor assoc comm id: none] .
TheavailableprintoptionsformasubsetoftheglobalprintoptionsdescribedinAppendixA.10,
which are ignored by this operation.
As an example, we can use metaPrettyPrint to pretty print the result of parsing at the
metalevel the list of tokens $ q q q in the module VENDING-MACHINE, first with prefix syntax,
then with mixfix syntax, and finally with mixfix syntax and taking into account the format
attribute.
Maude> reduce in META-LEVEL :
metaPrettyPrint(upModule(’VENDING-MACHINE, false), none,
’__[’$.Coin, ’__[’q.Coin, ’__[’q.Coin, ’q.Coin]]],
none) .
result NeQidList:
’__ ’‘( ’$ ’‘, ’__ ’‘( ’q ’‘, ’__ ’‘( ’q ’‘, ’q ’‘) ’‘) ’‘)
Maude> reduce in META-LEVEL :
metaPrettyPrint(upModule(’VENDING-MACHINE, false), none,
’__[’$.Coin, ’__[’q.Coin, ’__[’q.Coin, ’q.Coin]]],
mixfix) .
result NeTypeList: ’$ ’q ’q ’q
Maude> reduce in META-LEVEL :
metaPrettyPrint(upModule(’VENDING-MACHINE, false), none,
’__[’$.Coin, ’__[’q.Coin, ’__[’q.Coin, ’q.Coin]]],
mixfix format) .
result NeTypeList:
’\r ’\! ’$ ’\o ’\r ’\! ’q ’\o ’\r ’\! ’q ’\o ’\r ’\! ’q ’\o

<!-- manual page 474 -->
462CHAPTER 17. REFLECTION, METALEVEL COMPUTATION, AND INTERNAL STRATEGIES
Notice that the metaPrettyPrint operation uses the information provided by the format at-
tribute. For example, the operator $ in the module VENDING-MACHINE-SIGNATURE in Sec-
tion 5.1 was declared with attribute format (r! o), and therefore it is meta-pretty-printed as
’\r ’\! ’$ ’\o.
When the term contains variables, these are printed without their type annotation if and
only if they have been passed to the VariableSet argument of metaPrettyPrint.
Maude> reduce in META-LEVEL :
metaPrettyPrint(upModule(’VENDING-MACHINE, false), ’C2:Coin ; ’C3:Coin,
’__[’$.Coin, ’__[’C1:Coin, ’C2:Coin]],
mixfix format) .
result NeQidList: ’\r ’\! ’$ ’\o ’C1:Coin ’C2
metaPrintToString
ThemetaPrintToStringoperatorworkslikemetaPrettyPrintexceptthatitprintstoastring.
op metaPrintToString : Module VariableSet Term PrintOptionSet QidSet ~> String
[special (...)] .
The metaPrintToString operator is also a partial operation that takes as arguments the
metarepresentations of a module Rand of a term t together with a set of declared variables,
printing options, and a set of operator names whose arguments will be concealed. It returns
the string produced by pretty-printing the term t in the signature of R.
Below, we show the results for the same examples used for metaPrettyPrint() but using
metaPrintToString instead.
Maude> reduce in META-LEVEL :
metaPrintToString(upModule(’VENDING-MACHINE, false), none,
’__[’$.Coin, ’__[’q.Coin, ’__[’q.Coin, ’q.Coin]]],
none, none) .
result String: "__($, __(q, __(q, q)))"
Maude> reduce in META-LEVEL :
metaPrintToString(upModule(’VENDING-MACHINE, false), none,
’__[’$.Coin, ’__[’q.Coin, ’__[’q.Coin, ’q.Coin]]],
mixfix, none) .
result String: "$ q q q"
Maude> reduce in META-LEVEL :
metaPrintToString(upModule(’VENDING-MACHINE, false), none,
’__[’$.Coin, ’__[’q.Coin, ’__[’q.Coin, ’q.Coin]]],
mixfix format, none) .
result String:
"\033[31m\033[1m$\033[0m \033[31m\033[1mq\033[0m \033[31m\033[1mq\033[0m \033[31m\033[1mq\033[0m"
Maude> reduce in META-LEVEL :
metaPrintToString(upModule(’VENDING-MACHINE, false),
’C2:Coin ; ’C3:Coin,
’__[’$.Coin, ’__[’C1:Coin, ’C2:Coin]],
mixfix format, none) .
result String: "\033[31m\033[1m$\033[0m C1:Coin C2:Coin"

<!-- manual page 475 -->
17.6. THE META-LEVEL MODULE: METALEVEL OPERATIONS 463
metaParseStrategy
The function metaParseStrategy is the counterpart of metaParse for strategy expressions.
op metaParseStrategy : Module VariableSet QidList ~> Strategy? [special (...)] .
It takes the metarepresentation of a module along with a set of declared variables, and a list
of quoted identifiers. It tries to parse the given tokens as a strategy expression and return its
metarepresentation.
subsort Strategy < Strategy? .
op noStratParse : Nat -> Strategy? [ctor] .
op ambiguity : Strategy Strategy -> Strategy? [ctor] .
However, if there was no parse, it returns the term noStratParse(n), where n is the index
of the first bad token (counting from 0), or the number of tokens in the case of unexpected
end of input. If there are multiple parses, two of them are returned as an ambiguity(s1, s2)
term. As an example, we can parse the definition of the expand strategy in the QUEENS-STRAT
module:
Maude> red in META-LEVEL :
metaParseStrategy(upModule(’QUEENS-STRAT, false), none,
’top ’‘( ’next ’‘) ’;
’match ’L:List‘{Nat‘} ’such ’that ’isOk ’‘( ’L:List‘{Nat‘} ’‘)) .
rewrites: 2 in 6ms cpu (6ms real) (301 rewrites/second)
result Strategy:
top(’next[none]{empty}) ; match ’L:NatList s.t. (’isOk[’L:NatList] = ’true.Bool)
Instead of writing the sort of the variable explicitly as ’L:List‘\{Nat‘\} in the input, we may
pass a variable declaration for L to metaParseStrategy:
Maude> red metaParseStrategy(upModule(’QUEENS-STRAT, false), ’L:List‘{Nat‘},
’top ’‘( ’next ’‘) ’; ’match ’L ’such ’that ’isOk ’‘( ’L ’‘)) .
rewrites: 2 in 3ms cpu (4ms real) (300 rewrites/second)
result Strategy:
top(’next[none]{empty}) ; match ’L:NatList s.t. (’isOk[’L:NatList] = ’true.Bool)
metaPrettyPrintStrategy
The function metaPrettyPrintStrategy is the counterpart of metaPrettyPrint for strategy
expressions and the opposite of metaParseStrategy.
op metaPrettyPrintStrategy :
Module VariableSet Strategy PrintOptionSet ~> QidList [special (...)] .
It takes as arguments the metarepresentation of a module and of a strategy together with a set
of declared variables and printing options, and returns a list of quoted identifiers that represents
the string of tokens produced by pretty-printing the given strategy. The printing options are
the same as for metaPrettyPrint. For example, the pretty-printing of the application of the
rule next in QUEENS-STRAT can be obtained with:
Maude> red in META-LEVEL :
metaPrettyPrintStrategy(
upModule(’QUEENS-STRAT, false), none, ’next[none]{empty}, none) .
rewrites: 2 in 46ms cpu (45ms real) (43 rewrites/second)
result Sort: ’next

<!-- manual page 476 -->
464CHAPTER 17. REFLECTION, METALEVEL COMPUTATION, AND INTERNAL STRATEGIES
17.6.14 Sort operations
The META-LEVEL module also provides in a built-in way commonly needed operations on the
poset of sorts of a given module.
All these operations, related to sorts and kinds, take as first argument a term of sort Module.
Assumingthatthistermisindeedthemetarepresentationofamodule, theremainingarguments
might be terms representing sorts or kinds that do not correspond to sorts or kinds declared in
such a module; in this case, the operation is undefined.
In the following we include descriptions together with simple examples of using these oper-
ations.
sortLeq
The operation sortLeq takes as arguments the metarepresentation of a module Rand the
metarepresentations of two types, that is, either sorts or kinds.
op sortLeq : Module Type Type ~> Bool [special (...)] .
According to whether the types passed to sortLeq as arguments are metarepresented sorts
or kinds, we can distinguish the following cases:
• Assume first that both types given as arguments are two sorts sand s′. Let S be the set
of sorts in Rand let ≤Rbe its subsort relation. When s,s′∈S, sortLeq returns true if
s≤Rs′and false otherwise. For example,
Maude> reduce in META-LEVEL :
sortLeq(upModule(’NUMBERS, false), ’Zero, ’Nat) .
result Bool: true
Maude> reduce in META-LEVEL :
sortLeq(upModule(’NUMBERS, false), ’Zero, ’NzNat) .
result Bool: false
• If both types given as arguments are kinds in R, then sortLeq returns false when both
kinds are different and true when they are equal. For example,
Maude> reduce in META-LEVEL :
result Bool: true
sortLeq(upModule(’NUMBERS, false), ’‘[Zero‘], ’‘[Nat‘]) .
Maude> reduce in META-LEVEL :
result Bool: false
sortLeq(upModule(’NUMBERS, false), ’‘[Zero‘], ’‘[Bool‘]) .
• If the second argument is a sort in Rand the third one is a kind in R, then sortLeq
checks whether the given sort belongs to the given kind or not. If the second argument
is a kind and the third one is a sort, sortLeq always return false. For example,
Maude> reduce in META-LEVEL :
sortLeq(upModule(’NUMBERS, false), ’‘[Bool‘], ’Bool) .
result Bool: false
Maude> reduce in META-LEVEL :
result Bool: true
sortLeq(upModule(’NUMBERS, false), ’Zero, ’‘[NatSet‘]) .

<!-- manual page 477 -->
17.6. THE META-LEVEL MODULE: METALEVEL OPERATIONS 465
sameKind
The operation sameKind takes as arguments the metarepresentation of a module Rand the
metarepresentations of two types, that is, either sorts or kinds.
op sameKind : Module Type Type ~> Bool [special (...)] .
Let S be the set of sorts in Rand let ≤Rbe its subsort relation. When the two types passed
as arguments to sameKind are sorts s,s′∈S, the operation sameKind returns true if s and s′
belong to the same connected component in the subsort ordering ≤R, that is, if they belong
to the same kind, and false otherwise. When the two arguments are kinds in R, sameKind
returns true when they are indeed the same, and false otherwise. Finally, when one argument
is one sort and the other is a kind, this operation ckecks whether the sort belongs to the kind.
For example, we have the following reductions about sorts and kinds in the module NUMBERS.
Maude> reduce in META-LEVEL :
sameKind(upModule(’NUMBERS, false), ’Zero, ’NzNat) .
result Bool: true
Maude> reduce in META-LEVEL :
sameKind(upModule(’NUMBERS, false), ’Zero, ’Nat3) .
result Bool: false
Maude> reduce in META-LEVEL :
result Bool: true
sameKind(upModule(’NUMBERS, false), ’‘[Zero‘], ’‘[NzNat‘]) .
Maude> reduce in META-LEVEL :
result Bool: true
sameKind(upModule(’NUMBERS, false), ’‘[Zero‘], ’NzNat) .
completeName
The operation completeNametakes as arguments the metarepresentation of a module Rand the
metarepresentation of a sort sor a kind k. When its second argument is the metarepresentation
of a sort s, it returns the same metarepresentation of s. But if its second argument is the
metarepresentation of a kind k, then it returns the metarepresentation of the complete name
of k in R, i.e., the metarepresentation of the comma-separated list of the maximal elements of
the corresponding connected component.
op completeName : Module Type ~> Type [special (...)] .
For example,
Maude> reduce in META-LEVEL :
completeName(upModule(’NUMBERS, false), ’Zero) .
result Sort: ’Zero
Maude> reduce in META-LEVEL :
completeName(upModule(’NUMBERS, false), ’‘[Zero‘]) .
result Kind: ’‘[NatSeq‘,NatSet‘]

<!-- manual page 478 -->
466CHAPTER 17. REFLECTION, METALEVEL COMPUTATION, AND INTERNAL STRATEGIES
getKind and getKinds
The operation getKind takes as arguments the metarepresentation of a module Rand the
metarepresentation of a type, i.e., a sort or a kind. When its second argument is the metarep-
resentation of a type in R, it returns the metarepresentation of the complete name of the
corresponding kind.
op getKind : Module Type ~> Kind [special (...)] .
For example,
Maude> reduce in META-LEVEL :
getKind(upModule(’NUMBERS, false), ’Zero) .
result Kind: ’‘[NatSeq‘,NatSet‘]
Maude> reduce in META-LEVEL :
getKind(upModule(’NUMBERS, false), ’‘[Zero‘]) .
result Kind: ’‘[NatSeq‘,NatSet‘]
The operation getKinds takes as its only argument the metarepresentation of a module R
andreturnsthemetarepresentationofthesetofkindsdeclaredinR,withkindsmetarepresented
using their complete names.
op getKinds : Module ~> KindSet [special (...)] .
For example,
Maude> reduce in META-LEVEL : getKinds(upModule(’NUMBERS, false)) .
result NeKindSet: ’‘[Bool‘] ; ’‘[Nat3‘] ; ’‘[NatSeq‘,NatSet‘]
lesserSorts
The operation lesserSorts takes as arguments the metarepresentation of a module Rand the
metarepresentation of a type, i.e., a sort or a kind.
op lesserSorts : Module Type ~> SortSet [special (...)] .
Let S be the set of sorts in R. When s∈S, lesserSorts returns the metarepresentation
of the set of sorts strictly smaller than s in S. For example,
Maude> reduce in META-LEVEL :
lesserSorts(upModule(’NUMBERS, false), ’Nat) .
result NeSortSet: ’NzNat ; ’Zero
Maude> reduce in META-LEVEL :
lesserSorts(upModule(’NUMBERS, false), ’Zero) .
result EmptyTypeSet: (none).EmptyTypeSet
Maude> reduce in META-LEVEL :
lesserSorts(upModule(’NUMBERS, false), ’NatSeq) .
result NeSortSet: ’Nat ; ’NzNat ; ’Zero
When the second argument of lesserSorts metarepresents a kind in R, this operation
returns the metarepresentation of the set of all sorts in such kind. For example,
Maude> reduce in META-LEVEL :
lesserSorts(upModule(’NUMBERS, false), ’‘[NatSeq‘]) .
result NeSortSet: ’Nat ; ’NatSeq ; ’NatSet ; ’NzNat ; ’Zero
Maude> reduce in META-LEVEL :
lesserSorts(upModule(’NUMBERS, false), ’‘[Bool‘]) .
result Sort: ’Bool

<!-- manual page 479 -->
17.6. THE META-LEVEL MODULE: METALEVEL OPERATIONS 467
leastSort
The operation leastSort takes as arguments the metarepresentation of a module Rand the
metarepresentation of a term t, and it returns the metarepresentation of the least sort or kind
of tin R, obtained without reducing the term, that is, the memberships in the module are used
to get the information, but equations are not used to reduce the term.
op leastSort : Module Term ~> Type [special (...)] .
For example,
Maude> reduce in META-LEVEL :
leastSort(upModule(’NUMBERS, false), ’p[’s_[’zero.Zero]]) .
result Sort: ’Nat
glbSorts
The operation glbSorts takes as arguments the metarepresentation of a module Rand the
metarepresentations of two types, that is, either sorts or kinds.
op glbSorts : Module Type Type ~> TypeSet [special (...)] .
According to whether the types passed to glbSorts as arguments are metarepresented sorts
or kinds, we can distinguish the following cases:
• If both types given as arguments are sorts in R, then glbSorts returns the metarepre-
sentation of the set (which can be empty) consisting of the largest sorts that are common
subsorts of the two given sorts, that is, the set of maximal lower bounds of the two sorts;
when this set is a singleton set {s}, then s will be the greatest lower bound of the two
sorts, thus the operation name glbSorts.
For example, we have the following reductions concerning sorts in the module NUMBERS.
Maude> reduce in META-LEVEL :
glbSorts(upModule(’NUMBERS, false), ’Zero, ’Nat) .
result Sort: ’Zero
Maude> reduce in META-LEVEL :
glbSorts(upModule(’NUMBERS, false), ’NatSet, ’NatSeq) .
result Sort: ’Nat
Maude> reduce in META-LEVEL :
glbSorts(upModule(’NUMBERS, false), ’NzNat, ’NzNat) .
result Sort: ’NzNat
Maude> reduce in META-LEVEL :
glbSorts(upModule(’NUMBERS, false), ’Zero, ’NzNat) .
result EmptyTypeSet: (none).EmptyTypeSet
Maude> reduce in META-LEVEL :
glbSorts(upModule(’NUMBERS, false), ’NzNat, ’Bool) .
result EmptyTypeSet: (none).EmptyTypeSet
• If both types given as arguments are kinds in R, then glbSorts returns the empty
set when both kinds are different, and the metarepresentation of the kind (using the
corresponding complete name) when both kinds are equal. For example,
Maude> reduce in META-LEVEL :
glbSorts(upModule(’NUMBERS, false), ’‘[Nat‘], ’‘[Bool‘]) .
result EmptyTypeSet: (none).EmptyTypeSet

<!-- manual page 480 -->
468CHAPTER 17. REFLECTION, METALEVEL COMPUTATION, AND INTERNAL STRATEGIES
Maude> reduce in META-LEVEL :
glbSorts(upModule(’NUMBERS, false),’‘[Nat‘],’‘[NatSeq‘]) .
result Kind: ’‘[NatSeq‘,NatSet‘]
• If one type is one sort in Rand the other one is a kind in R, then glbSorts returns
the metarepresentation of the sort when the sort belongs to the kind, and the empty set
otherwise. For example,
Maude> reduce in META-LEVEL :
glbSorts(upModule(’NUMBERS, false), ’‘[Nat‘], ’Bool) .
result EmptyTypeSet: (none).EmptyTypeSet
Maude> reduce in META-LEVEL :
result Sort: ’Zero
glbSorts(upModule(’NUMBERS, false), ’‘[NatSeq‘], ’Zero) .
Maude> reduce in META-LEVEL :
result Sort: ’NzNat
glbSorts(upModule(’NUMBERS, false), ’NzNat, ’‘[NatSet‘]) .
maximalSorts and minimalSorts
The operations maximalSorts and minimalSorts take as arguments the metarepresentation of
a module Rand the metarepresentation of a kind k. If k is a kind in R, maximalSorts returns
the metarepresentation of the set of the maximal sorts in the connected component of k, while
minimalSorts returns the metarepresentation of the set of its minimal sorts.
op maximalSorts : Module Kind ~> SortSet [special (...)] .
op minimalSorts : Module Kind ~> SortSet [special (...)] .
For example,
Maude> reduce in META-LEVEL :
maximalSorts(upModule(’NUMBERS, false), ’‘[Zero‘]) .
result NeSortSet: ’NatSeq ; ’NatSet
Maude> reduce in META-LEVEL :
minimalSorts(upModule(’NUMBERS, false), ’‘[Zero‘]) .
result NeSortSet: ’Zero ; ’NzNat
maximalAritySet
The operation maximalAritySet takes as arguments the metarepresentation of a module R,
the metarepresentation of an operator f in R, the metarepresentation of an arity (list of types)
for f and the metarepresentation of a sort s, and then computes the set of maximal arities (lists
of types) that f could take and have a sort s′≤Rs. This result might be the empty set if s is
small or f is only defined at the kind level.
Notice that the result of this operation is a set of lists of types, which is built by means
of the following syntax, extending the syntax for building lists of types that we only show
partially here and whose full specification can be found in the module META-MODULE in the file
prelude.maude available with the Maude distribution.
sort NeTypeList TypeList .
op nil : -> TypeList [ctor] .
op __ : TypeList TypeList -> TypeList [ctor ditto] .

<!-- manual page 481 -->
17.6. THE META-LEVEL MODULE: METALEVEL OPERATIONS 469
sort TypeListSet .
subsort TypeList TypeSet < TypeListSet .
op _;_ : TypeListSet TypeListSet -> TypeListSet [ctor ditto] .
eq T:TypeList ; T:TypeList = T:TypeList .
op maximalAritySet : Module Qid TypeList Sort ~> TypeListSet
[special (...)] .
Let us consider for example the operator _+_ in the module NUMBERS, where it is overloaded
by means of the following declarations:
op _+_ : Nat Nat -> Nat [assoc comm].
op _+_ : NzNat Nat -> NzNat [ditto] .
op _+_ : Nat3 Nat3 -> Nat3 [comm] .
With this information, we obtain the following reductions concerning this operator:
Maude> reduce in META-LEVEL :
maximalAritySet(upModule(’NUMBERS, false),
’_+_, ’NzNat ’NzNat, ’NzNat) .
result TypeListSet: ’Nat ’NzNat ; ’NzNat ’Nat
Maude> reduce in META-LEVEL :
maximalAritySet(upModule(’NUMBERS, false),
’_+_, ’Nat ’Nat, ’NzNat) .
result TypeListSet: ’Nat ’NzNat ; ’NzNat ’Nat
Maude> reduce in META-LEVEL :
maximalAritySet(upModule(’NUMBERS, false),
’_+_, ’Nat ’Nat, ’Nat) .
result NeTypeList: ’Nat ’Nat
Maude> reduce in META-LEVEL :
maximalAritySet(upModule(’NUMBERS, false),
’_+_, ’Nat3 ’Nat3, ’Nat3) .
result NeTypeList: ’Nat3 ’Nat3
Notice that if the operator f and the list of types passed as arguments to maximalAritySet
do not match, then the result is an error, which is represented as a non-reduced term in a
metalevel kind. We have for instance the following example where we have omitted the lengthy
metarepresentation of the NUMBERS module.
Maude> reduce in META-LEVEL :
maximalAritySet(upModule(’NUMBERS, false),
’_+_, ’Nat3 ’Nat3, ’NzNat) .
result [GTermList,ParameterList,QidList,
TypeListSet,Type?,ModuleExpression,Header]:
maximalAritySet(fmod ’NUMBERS is ... endfm,
’_+_, ’Nat3 ’Nat3, ’NzNat)
17.6.15 Other metalevel operations: wellFormed
The operation wellFormed can take as arguments the metarepresentation of a module R, or
the metarepresentation of a module Rand a term t, or the metarepresentation of a module
Rand a substitution σ. In the first case, it returns true if Ris a well-formed module, and

<!-- manual page 482 -->
470CHAPTER 17. REFLECTION, METALEVEL COMPUTATION, AND INTERNAL STRATEGIES
false otherwise. In the second case, if tis a well-formed term in R, it returns true; otherwise,
it returns false. Finally, in the third case, if σ is a well-formed substitution in R, it returns
true; otherwise, it returns false.
op wellFormed : Module -> Bool [special (...)] .
op wellFormed : Module Term ~> Bool [special (...)] .
op wellFormed : Module Substitution ~> Bool [special (...)] .
Note that the first operation is total, while the other two are partial (notice the form of
the arrow in the declarations). The reason is that the last two are not defined when the term
given as first argument does not represent a module, and then it does not make sense to check
whether a term or substitution is well formed with respect to such a wrong “module.” For
example,
Maude> reduce in META-LEVEL :
wellFormed(upModule(’NUMBERS, false)) .
result Bool: true
Maude> reduce in META-LEVEL :
wellFormed(upModule(’NUMBERS, false), ’p[’zero.Zero]) .
result Bool: true
Maude> reduce in META-LEVEL :
wellFormed(upModule(’NUMBERS, false),
’s_[’zero.Zero, ’zero.Zero]) .
Advisory: could not find an operator s_ with appropriate domain
in meta-module NUMBERS when trying to interprete metaterm
’s_[’zero.Zero,’zero.Zero].
result Bool: false
Maude> reduce in META-LEVEL :
wellFormed(upModule(’NUMBERS, false),
’N:Zero <- ’zero.Zero) .
result Bool: true
Maude> reduce in META-LEVEL :
wellFormed(upModule(’NUMBERS, false),
’N:Nat <- ’p[’zero.Zero]) .
result Bool: false
Maude> reduce in META-LEVEL :
wellFormed(upModule(’NUMBERS, false),
’N:Zero <- ’s_[’zero.Zero,’zero.Zero]) .
Advisory: could not find an operator s_ with appropriate domain
in meta-module NUMBERS when trying to interprete metaterm
’s_[’zero.Zero,’zero.Zero].
result Bool: false
17.7 Internal strategies
System modules in Maude are rewrite theories that do not need to be Church-Rosser and
terminating. Therefore, we need to have good ways of controlling the rewriting inference

<!-- manual page 483 -->
17.7. INTERNAL STRATEGIES 471
process—which in principle could not terminate or go in many undesired directions—by means
of adequate strategies, as already explained in Chapter 10.
In Maude, thanks to its reflective capabilities, strategies can be made internal to the system.
Thatis, theycanbedefinedusingstatementsinanormalmoduleinMaude, andcanbereasoned
about as with statements in any other module. In general, internal strategies are defined in
extensions of the META-LEVEL module by using metaReduce, metaApply, metaXapply, etc., as
building blocks.
We illustrate some of these possibilities by implementing the following strategies for con-
trolling the execution of the rules in the VENDING-MACHINE module in Section 5.1:
1. insert either a dollar or a quarter in the vending machine;
2. only buy cakes, and buy as many cakes as possible, with the coins already inserted;
3. only buy either cakes or apples, and buy at most n of them, with the coins already
inserted;
4. buy the same number of apples and cakes, and buy as many as possible, with the coins
already inserted.
Alongside each internal strategy we also show an equivalent strategy expression in the Maude
strategy, explained in Chapter 10. Consider the module BUYING-STRATS below, which imports
the META-LEVEL module.
fmod BUYING-STRATS is
protecting META-LEVEL .
The function insertCoin below defines the strategy (1): it expects as first argument either
’add-q or ’add-$, for inserting a quarter or a dollar, respectively, and as second argument
the metarepresentation of the marking of a vending machine, and it applies once the rule
corresponding to the given label. The rules add-q and add-$ are applied using the descent
function metaXapply. A rule cannot be applied when the result of metaXapply-ing the rule
is not a term of sort Result4Tuple. Note the use of a matching equation in the condition to
simplify the presentation of the righthand side of the equation (see Section 4.3), as well as the
use of the statement attribute owise (see Section 4.5.4) to define the function insertCoin for
unexpected cases.
var T : Term .
var Q : Qid .
var N : Nat .
vars BuyItem? BuyCake? Change? : [Result4Tuple].
op insertCoin : Qid Term -> Term .
ceq insertCoin(Q, T)
= if BuyItem? :: Result4Tuple
then getTerm(BuyItem?)
else T
fi
if (Q == ’add-q or Q == ’add-$)
/\ BuyItem? := metaXapply(upModule(’VENDING-MACHINE, false),
T, Q, none, 0, unbounded, 0) .
eq insertCoin(Q, T) = T [owise] .
Using the object-level strategy language, the two possibilities of the insertCoin strategy are
simply represented by the rule-application expressions add-q and add-$.
The function onlyCakes below defines the strategy (2): it applies the rule buy-c as many
times as possible, applying the rule change whenever it is necessary. In particular, if the rule

<!-- manual page 484 -->
472CHAPTER 17. REFLECTION, METALEVEL COMPUTATION, AND INTERNAL STRATEGIES
buy-c can be applied, then there is a recursive call to the function onlyCakes with the term
resulting from its application. If the rule buy-c cannot be applied, then the application of the
rule change is attempted. If the rule change can be applied, then there is a recursive call to
the function onlyCakes with the term resulting from the change rule application. Otherwise,
the argument is returned unchanged. The rules buy-c and change are also applied using the
descent function metaXapply.
op onlyCakes : Term -> Term .
ceq onlyCakes(T)
= if BuyCake? :: Result4Tuple
then onlyCakes(getTerm(BuyCake?))
else (if Change? :: Result4Tuple
then onlyCakes(getTerm(Change?))
else T
fi)
fi
if BuyCake? := metaXapply(upModule(’VENDING-MACHINE, false),
T, ’buy-c, none, 0, unbounded, 0)
/\ Change? := metaXapply(upModule(’VENDING-MACHINE, false),
T, ’change, none, 0, unbounded, 0) .
This is equivalent to the expression (buy-c or-else change) ! in the Maude strategy lan-
guage.
The function onlyNitems defines the strategy (3): it applies either the rule buy-c or buy-a
(but not both) at most n times. As expected, the rules are applied using the descent function
metaXapply. Notetheuseofthesymmetricdifferenceoperatorsd(seeSection8.3)todecrement
N.
op onlyNitems : Term Qid Nat -> Term .
ceq onlyNitems(T, Q, N)
= if N == 0
then T
else (if BuyItem? :: Result4Tuple
then onlyNitems(getTerm(BuyItem?), Q, sd(N, 1))
else (if Change? :: Result4Tuple
then onlyNitems(getTerm(Change?), Q, N)
else T
fi)
fi)
fi
if (Q == ’buy-c or Q == ’buy-a)
/\ BuyItem? := metaXapply(upModule(’VENDING-MACHINE, false),
T, Q, none, 0, unbounded, 0)
/\ Change? := metaXapply(upModule(’VENDING-MACHINE, false),
T, ’change, none, 0, unbounded, 0) .
eq onlyNitems(T, Q, N) = T [owise] .
Intheobject-levelstrategylanguage, onlyNitemscanbedescribedusingthefollowingrecursive
strategy definition:
smod BUYING-STRAT-LANG is
protecting VENDING-MACHINE .
protecting NAT .

<!-- manual page 485 -->
17.7. INTERNAL STRATEGIES 473
strat onlyNitems : Item Nat @ Marking .
strat oneItem : Item @ Marking .
var I : Item . var N : Nat .
sd onlyNitems(I, 0) := idle .
sd onlyNitems(I, s N) := (oneItem(I) or-else change) ; onlyNitems(I, N) .
sd oneItem(c) := buy-c .
sd oneItem(a) := buy-a .
endsm
Finally, the function cakesAndApples defines the strategy (4): it applies the rule buy-c as
many times as the rule buy-a. To define this function, we use an auxiliary Boolean function
buyItem? that determines whether a given rule (buy-c or buy-a) can be applied. In the
definition of cakesAndApples the Boolean function buyItem? is used to check if the rule
buy-a can be applied after applying the rule buy-c. When the answer is true, then buy-c and
buy-a are applied once, using the function onlyNitems with the appropriate arguments, and
the function cakesAndApples is applied again to the result.
op cakesAndApples : Term -> Term .
op buyItem? : Term Qid -> Bool .
ceq buyItem?(T, Q)
= if BuyItem? :: Result4Tuple
then true
else (if Change? :: Result4Tuple
then buyItem?(getTerm(Change?), Q)
else false
fi)
fi
if (Q == ’buy-c or Q == ’buy-a)
/\ BuyItem? := metaXapply(upModule(’VENDING-MACHINE, false),
T, Q, none, 0, unbounded, 0)
/\ Change? := metaXapply(upModule(’VENDING-MACHINE, false),
T, ’change, none, 0, unbounded, 0) .
eq buyItem?(T, Q) = false [owise] .
eq cakesAndApples(T)
= if buyItem?(T, ’buy-a)
then (if buyItem?(onlyNitems(T, ’buy-a, 1), ’buy-c)
then cakesAndApples(onlyNitems(onlyNitems(T, ’buy-a, 1),
’buy-c, 1))
else T
fi)
else T
fi .
endfm
In the Maude strategy language, the cakesAndApples strategy can be written as follows by
reusing the already defined onlyNitems strategy.
(onlyNitems(a, 1) ; onlyNitems(c, 1)) !

<!-- manual page 486 -->
474CHAPTER 17. REFLECTION, METALEVEL COMPUTATION, AND INTERNAL STRATEGIES
In effect, exactly one apple and one cake are bought in each iteration of the normalization !
operator. When one of the onlyNitems calls fails, the whole iteration will fail too, and the
result of the previous step will be returned as a result.
As examples, we apply below the buying strategies (2–4) to spend in different ways the
same amount of money: three dollars and a quarter.
Maude> reduce in BUYING-STRATS :
onlyCakes(’__[’$.Coin, ’$.Coin, ’$.Coin, ’q.Coin]) .
result GroundTerm: ’__[’q.Coin, ’c.Item, ’c.Item, ’c.Item]
Maude> reduce in BUYING-STRATS :
onlyNitems(’__[’$.Coin, ’$.Coin, ’$.Coin, ’q.Coin],
’buy-a, 3) .
result GroundTerm:
’__[’q.Coin, ’q.Coin, ’q.Coin, ’q.Coin, ’a.Item, ’a.Item, ’a.Item]
Maude> reduce in BUYING-STRATS :
cakesAndApples(’__[’$.Coin, ’$.Coin, ’$.Coin, ’q.Coin]) .
result GroundTerm: ’__[’$.Coin, ’q.Coin, ’q.Coin, ’a.Item, ’c.Item]
ThesameexamplescanbereproducedwiththeequivalentstrategyexpressionsintheMaude
strategy language that we have introducing together with the internal ones.
Maude> srewrite in VENDING-MACHINE : $ $ $ q
using (buy-c or-else change) ! .
Solution 1
result Marking: q c c c
No more solutions.
Maude> srewrite in BUYING-STRAT-LANG : $ $ $ q using onlyNitems(a, 3) .
Solution 1
result Marking: q q q q a a a
No more solutions.
Maude> srewrite in BUYING-STRAT-LANG : $ $ $ q
using (onlyNitems(a, 1) ; onlyNitems(c, 1)) ! .
Solution 1
result Marking: $ q q a c
No more solutions.
There is in fact great freedom for defining many different types of strategies, or even many
different strategy languages inside Maude. As illustrated above with simple examples, this can
be done in a completely user-definable way, so that users are not limited by a fixed and closed
particular strategy language. See [20] for a general methodology for defining internal strategy
languages using reflection, and [21, 23] for other examples of rewriting strategies defined in
Maude.
However, the great freedom of defining internal strategies at the metalevel is purchased at
some cost. First, some familiarity with Maude’s metalevel features is required; and second,

<!-- manual page 487 -->
17.7. INTERNAL STRATEGIES 475
some cost in performance is incurred in comparison with what might be possible in a direct
implementation using Maude’s rewrite engine. To address these two issues, the Maude strategy
language, which can be used entirely at the object level, has been designed and implemented,
as described in Chapter 10. In the next section, we further discuss the relation between internal
strategies and the Maude strategy language.
17.7.1 Internal and object-level strategies
After the addition of the object-level strategy language to Maude, internal strategies may seem
nolongerneededforprogrammingrewritingstrategies. Indeed, moststrategiescanbeexpressed
more clearly and executed more easily at the object level. However, the greater freedom of re-
flective programs may make them sometimes more convenient, despite their intrinsic difficulties.
Expressions in the strategy language can only modify a term through the application of rewrite
rules defined in the module where they are evaluated. Hence, describe proper rewriting strate-
gies on the underlying rewrite system by construction. This is not automatically guaranteed
by metalevel programs, and it is the responsibility of the programmer to take care of it.
In general, internal strategies can be naturally translated to the object-level language and
vice versa, as shown with the previous examples. Rule application expressions are analogous to
thedescentfunctionsmetaApplyandmetaXapply, matchoperatorsareequivalenttometaMatch
and metaXmatch, recursive strategy definitions behave like recursive metalevel functions, and
the other control structures of the strategy language can also be implemented at the metalevel.
Nevertheless, while the search and enumeration of the results is implicit in the strategy lan-
guage, it has to be explicitly implemented within the internal strategies, and this is a source of
additional complexity. On the other hand, some strategies that can be easily programmed at
the metalevel are not so easily expressed in the strategy language. A non-exhaustive list of use
cases is the following:
• Generic strategies are better expressed reflectively, since strategy expressions are in prin-
ciple specific to the module where they are defined. Moreover, the strategy language does
not allow traversing terms generically, without explicit mention to a particular signature.
Given a specification of the λ-calculus, implementing a strategy to apply the β-reduction
rule in the innermost rightmost redex of a λ-term is easy using the matchrew operator of
the strategy language. However, it is impossible to write a generic version of this strat-
egy, valid for any module and any term, which is rather straightforward using an internal
strategy.
op innerRightmost : Module Term ~> ResultTriple? .
op innerRightmost : Module TermList TermList ~> ResultTriple? .
var M : Module . vars TL TL’ : TermList . vars Ty Ty’ : Type .
vars T T’ B : Term .
var Sb : Substitution .
ceq innerRightmost(M, Q[TL]) = {Q[TL’], Ty, Sb}
if {TL’, Ty, Sb} := innerRightmost(M, TL, empty) .
eq innerRightmost(M, T) = metaApply(M, T, ’anyRule, none, 0) [owise] .
ceq innerRightmost(M, (TL, T), TL’) = {(TL, T’, TL’), Ty, Sb}
if {T’, Ty, Sb} := innerRightmost(M, T) .
eq innerRightmost(M, (TL, T), TL’) =
innerRightmost(M, TL, (T, TL’)) .

<!-- manual page 488 -->
476CHAPTER 17. REFLECTION, METALEVEL COMPUTATION, AND INTERNAL STRATEGIES
Nevertheless,sometechnicaldifficultiesareinvolvedinthisdefinition,sinceneithermetaRewrite
nor metaXapply can be directly used to execute a single rewrite with any rule in the mod-
ule. We assume that the module M has been transformed to label or relabel every rule
with anyRule. Alternatively, using the metarepresentation of the strategy language (see
Section 17.3), we can automatically generate strategy expressions that particularize a
generic strategy on a given module by a reflective theory transformation [111].
• Strategies in the object-level language are always applied to single terms and the results
of a strategy can never be seen as a whole. For example, a local optimization strategy
may need to look at all solutions of a substrategy and select the “best” one to continue
with. This is not possible in the Maude strategy language, but it will be relatively easy
to achieve at the metalevel, where we can examine all the solutions of the substrategy.
In fact, mixing both worlds, we can write an internal strategy that selects the greatest
solution of a given strategy expression for an assumed < operator:
op bestSolution : Module Term Strategy ~> ResultPair? .
op bestSolution : Module Term Strategy ResultPair Nat ~> ResultPair? .
var S : Strategy .
ceq bestSolution(M, T, S) = bestSolution(M, T, S, {T’, Ty}, 1)
if {T’, Ty} := metaSrewrite(M, T, S, breadthFirst, 0) .
ceq bestSolution(M, T, S) = failure
if metaSrewrite(M, T, S, breadthFirst, 0) == failure .
ceq bestSolution(M, T, S, {B, Ty}, N) = bestSolution(M, T, S,
if getTerm(metaReduce(M, ’_<_[B, T’])) == ’true.Bool
then {T’, Ty’} else {B, Ty} fi, s N)
if {T’, Ty’} := metaSrewrite(M, T, S, breadthFirst, N) .
ceq bestSolution(M, T, S, {B, Ty}, N) = {B, Ty}
if metaSrewrite(M, T, S, breadthFirst, N) == failure .
• Strategy combinators and definitions cannot return any information to their users in ad-
dition to the rewritten term, even though this is standard in metalevel programs. A
reflective strategy is able to recover the matching context and substitution after applying
a rule with metaXapply, while the strategy language programmer should find an indi-
rect method to obtain this information if required. Similarly, strategy calls take input
arguments, but they do not provide any additional output. This inconvenience can be
overcome by making strategy definitions pass the required data to the next strategy by a
final recursive call with the appropriate arguments. For example, a hypothetical sequence
of two strategy calls x′ <- st1(x) ; st2(y, x′) can be transformed by extending the
input arguments of st1 with yand calling st2(y, x′) at the end of every control path in
its definitions. However, this compromises the modularity and reusability of the strate-
gies.

