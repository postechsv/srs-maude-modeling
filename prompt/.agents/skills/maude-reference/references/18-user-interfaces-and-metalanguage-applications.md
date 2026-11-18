# Chapter 18: User Interfaces and Metalanguage Applications (manual pages 489-504)

<!-- manual page 489 -->
Chapter 18
User Interfaces and Metalanguage
Applications
This chapter explains how to use the facilities provided by the predefined modules META-LEVEL
(Chapter 17), STD-STREAM (Section 9.1), LOOP-MODE (Section 18.4), and LEXICAL (Section 8.12)
for constructing user interfaces and metalanguage applications, in which Maude is used not only
to define a domain-specific language or tool, but also to build an environment for the given
language or tool. In such applications, the STD-STREAMand the LOOP-MODEmodules can be used
to handle the input/output and to maintain the persistent state of the language environment
or tool.
18.1 User interfaces
InordertogenerateinMaudeaninterface foranapplicationP,thefirstthingweneedtodoisto
define the language for interaction. This can be done by defining the datatype of its commands
and other constructs by means of a signature SignP. Then, we can use either STD-STREAM
or LOOP-MODE to define the interaction through the terminal, using standard input/output
facilities. In this first section, we use the facilities provided by STD-STREAM. The same example
will also be used later in Section 18.4 to illustrate the use of the LOOP-MODE module.
As a running example for this chapter, we will specify a basic interface for the vending
machine introduced in Section 5.1. First, we define in the module VENDING-MACHINE-GRAMMAR
a language for interacting with the vending machine. The signature of this module extends
the signature of VENDING-MACHINE-SIGNATURE with operators to represent the valid actions,
namely: insert $ and insert q for inserting a dollar or a quarter in the machine; show
basket and show credit for showing the items already bought or the remaining credit; and
buy__(s) for buying a number of pieces of the same item.
fmod VENDING-MACHINE-GRAMMAR is
protecting VENDING-MACHINE-SIGNATURE .
protecting NAT .
sort Action .
op insert $ : -> Action .
op insert q : -> Action .
op show basket : -> Action .
op show credit : -> Action .
op buy__(s) : Nat Item -> Action .
477

<!-- manual page 490 -->
478 CHAPTER 18. USER INTERFACES AND METALANGUAGE APPLICATIONS
endfm
The interaction with the standard IO streams is performed using objects and messages.
The next step is therefore to define the objects that will keep the state of the system and will
interact with the IO streams. Our goal is to have a loop between the system and the user in
which the vending machine asks for input, and then, depending on the specific action, performs
one task or another, and gives its response to the user.
In the module VENDING-MACHINE-IO below, we introduce the necessary declarations to use
objects of a class VM with attributes action, to keep the last action requested from the user
(inserting a coin, showing information about the remaining credit or the items already bought,
or buying one or more items), and marking, to keep the state of the machine (the marking
of the vending machine, that is, the remaining credit plus the items already bought). The
constants vm and idle are introduced for convenience to denote the identifier of the vending
machine object and to denote the idle action in which the system is waiting for input from the
user once the actual action is processed.
omod VENDING-MACHINE-IO is
including STD-STREAM .
including VENDING-MACHINE-GRAMMAR .
protecting BUYING-STRATS .
protecting CONVERSION .
protecting LEXICAL .
class VM | action : Action, marking : Marking .
op idle : -> Action .
op vm : -> Oid .
To operate on the system we introduce two further constants: VM-GRAMMAR represents the
moduleinwhichtheinputfromtheuserisgoingtobeparsed,theaboveVENDING-MACHINE-GRAMMAR;
the vending machine configuration is used to initialize the system, and it includes the portal
to interact with standard streams, the vm object, initially with null marking and action idle,
and a first write message to show the user that the machine is started.
op VM-GRAMMAR : -> Module .
eq VM-GRAMMAR = upModule(’VENDING-MACHINE-GRAMMAR, false) .
op vending machine : -> Configuration .
eq vending machine
= <>
< vm : VM | marking : null, action : idle >
write(stdout, vm, "\n\t Vending machine\n") .
The interaction with the user is then handled by the following two rules. Once a write
message is processed, the output stream sends back a wrote message, which is used to request
an action from the user, who is prompted with a > character. When the user hits the return
key, the stream object sends the input in a gotLine message. In the rule below, we expect
either a "quit" that will finish the interaction, a valid input in the VM-GRAMMAR grammar, or a
non-valid input. In the first case, a goodbye message is shown and the interaction terminates
(note that the object is removed from the state). If there is an invalid input, an error message
is shown and the interaction continues. If the input is valid in the VM-GRAMMAR grammar, the
corresponding action is placed in the action attribute of the VM object for further processing.
Note the use of the tokenize function1 to tokenize the input, which is received as a string,
1The tokenize function tokenizes strings into lists of quoted identifiers (see Section 8.12).

<!-- manual page 491 -->
18.1. USER INTERFACES 479
before parsing it. The operation metaParse checks whether the input stream corresponds to a
term of sort Action. If this is the case, metaParse returns the metarepresentation of that term,
which is then “moved down” using the META-LEVEL function downTerm (see Section 17.6.1).
vars O O’ : Oid .
var Str : String .
rl < O : VM | >
wrote(O, O’)
=> < O : VM | >
getLine(stdin, O, "> ") .
rl < O : VM | action : idle >
gotLine(O, O’, Str)
=> if tokenize(Str) == ’quit
then write(stdout, O, "goodbye\n")
else if metaParse(VM-GRAMMAR, tokenize(Str), ’Action) :: ResultPair
then < O : VM |
action : downTerm(
getTerm(
metaParse(VM-GRAMMAR, tokenize(Str), ’Action)),
idle) >
else < O : VM | action : idle >
write(stdout, O, "Invalid input\n")
fi
fi .
The processing of the valid inputs, the actions, is performed by several rewriting rules,
modeling the changes produced in the state of the vending machine by the actions requested by
the client. To define the interaction of the state of the vending machine with the client, we can
use the strategies introduced in the BUYING-STRATS module described in Section 17.7. Recall
that BUYING-STRATS includes the META-LEVEL module.
var A : Action .
var I : Item .
var C : Coin .
var M : Marking .
vars QIL QIL’ QIL’’ : QidList .
var N : Nat .
For the show basket and show credit actions, the following rules extract the informa-
tion about the remaining credit or the items already bought, and send a write message with
the corresponding information. In the definitions of the auxiliary functions showBasket and
showCredit, the operation metaPrettyPrinttakes the metarepresentation of a coin or an item,
and returns the list of quoted identifiers that encode the list of tokens produced by pretty-
printing the coin or the item in the module VENDING-MACHINE-SIGNATURE. Coins and items,
and, more generally, markings of a vending machine are metarepresented using the META-LEVEL
function upTerm (see Section 17.6.1).
op showBasket : Marking -> QidList .
eq showBasket(I M)
= metaPrettyPrint(upModule(’VENDING-MACHINE-SIGNATURE, false),
upTerm(I))
showBasket(M) .
eq showBasket(C M) = showBasket(M) .

<!-- manual page 492 -->
480 CHAPTER 18. USER INTERFACES AND METALANGUAGE APPLICATIONS
eq showBasket(null) = nil .
op showCredit : Marking -> QidList .
eq showCredit(C M)
= metaPrettyPrint(upModule(’VENDING-MACHINE-SIGNATURE, false),
upTerm(C))
showCredit(M) .
eq showCredit(I M) = showCredit(M) .
eq showCredit(null) = nil .
rl < O : VM | action : show basket, marking : M >
=> < O : VM | action : idle, marking : M >
write(stdout, O, "basket: " + printTokens(showBasket(M)) + "\n") .
rl < O : VM | action : show credit, marking : M >
=> < O : VM | action : idle, marking : M >
write(stdout, O, "credit: " + printTokens(showCredit(M)) + "\n") .
The following rules implement the actions of inserting a dollar or a quarter in the vending
machine. The strategy insertCoin defined in the module BUYING-STRATS (see Section 17.7)
is used to produce the corresponding change in the current marking of the vending machine.
Since strategies are applied at the metalevel, both the marking of the vending machine and the
coin to be inserted must be first metarepresented using again the META-LEVEL function upTerm.
rl < O : VM | action : insert q, marking : M >
=> < O : VM | action : idle,
marking : downTerm(insertCoin(’add-q, upTerm(M)), null) >
write(stdout, O, "one quarter introduced\n") .
rl < O : VM | action : insert $, marking : M >
=> < O : VM | action : idle,
marking : downTerm(insertCoin(’add-$, upTerm(M)), null) >
write(stdout, O, "one dollar introduced\n") .
The last two rules implement the actions of buying one or more items. The strategy
onlyNitems defined in the module BUYING-STRATS (see Section 17.7) is used to produce the
corresponding change in the current marking of the vending machine. Again, since strategies
are applied at the metalevel, the marking of the vending machine must be first metarepresented.
rl < O : VM | action : (buy N c (s)), marking : M >
=> < O : VM | action : idle,
marking : downTerm(onlyNitems(upTerm(M), ’buy-c, N), null) >
write(stdout, O, string(N, 10) + " cakes bought\n") .
rl < O : VM | action : (buy N a (s)), marking : M >
=> < O : VM | action : idle,
marking : downTerm(onlyNitems(upTerm(M), ’buy-a, N), null) >
write(stdout, O, string(N, 10) + " apples bought\n") .
endom
18.2 The interaction with the system
Withtheabovedefinitions,wecannowillustratethebasicinteractionwiththevendingmachine.
Once the VENDING-MACHINE-IO module has been entered, we can initiate the execution using

<!-- manual page 493 -->
18.3. TOKENS, BUBBLES, AND METAPARSING 481
the erewrite command and the vending machine constant.
Maude> erewrite in VENDING-MACHINE-IO : vending machine .
Vending machine
Once the machine interface has been initialized, we can input any data by writing it after
the prompt. For example,
> insert $
one dollar introduced
> show credit
credit: $
> insert $
one dollar introduced
> insert q
one quarter introduced
> buy 1 apple(s)
Invalid input
> buy 1 a(s)
1 apples bought
> show basket
basket: a
> show credit
credit: $ q q
> insert $
one dollar introduced
> buy 3 a(s)
3 apples bought
> show basket
basket: a a a a
> show credit
credit: q
We can terminate the interface by introducing the quit action.
> quit
goodbye
18.3 Metalanguage applications: tokens, bubbles, and
metaparsing
The example presented in the previous two sections is a toy example to illustrate the basic fea-
tures of a possible interaction loop with the system. However, the most interesting applications
of the STD-STREAM module are metalanguage applications, in which Maude is used to define
the syntax, parse, execute, and pretty print the execution results of a given object language or
tool. In such applications, most of the hard work is done by the META-LEVEL module, but we
can use a similar approach to handle the input/output and maintaining the persistent state of
the object language interpreter or tool.
In order to generate in Maude an environment for a language L, including the case of
a language with user-definable syntax, the first thing we need to do is to define the syntax
for L-modules. This can be done by defining a datatype for L-modules, as well as auxiliary
declarations for commands and other constructs, by means of a signature SignL. Maude

<!-- manual page 494 -->
482 CHAPTER 18. USER INTERFACES AND METALANGUAGE APPLICATIONS
s s s 0 provides great flexibility to do this, thanks to its mixfix front-end and to the use of bubbles (any
non-empty list of Maude identifiers). The intuition behind bubbles is that they correspond to
pieces of a module in a language that can only be parsed once the grammar introduced by the
signature of the module is available.
The idea is that, for a language that allows modules with user-definable syntax—as it is the
case for Maude itself—it is natural to see its syntax as a combined syntax at two different levels:
(1) what we may call the top-level syntax of the language, and (2) the user-definable syntax
introduced in each module. The bubble datatype allows us to reflect this duality of levels in
the syntax definition by encapsulating portions of (as yet unparsed) text in the user-definable
syntax. Similar ideas have been exploited using ASF+SDF [34, 35].
To illustrate this concept, suppose that we want to define the syntax of Maude in Maude.
Consider the following Maude module:
fmod NAT3 is
sort Nat3 .
op s_ : Nat3 -> Nat3 .
op 0 : -> Nat3 .
eq = 0 .
endfm
Notice that the lists of characters inside the boxes are not part of the top-level syntax of
Maude and therefore should be treated as bubbles until they are parsed. In fact, they can only
be parsed with the grammar associated with the signature of the module NAT3. In this sense,
we say that the syntax for Maude modules is a combination of two levels of syntax. The term
s s s 0, for example, has to be parsed in the grammar associated with the signature of NAT3.
The definition of the syntax of Maude in Maude must reflect this duality of syntax levels.
So far, we have talked about bubbles in a generic way. In fact, there can be many different
kinds of bubbles. In Maude we can define different types of bubbles as built-in datatypes by
parameterizing their definition. Thus, for example, a bubble of length one, which we call a
token, can be defined as follows:
sort Token .
op token : Qid -> Token
[special (id-hook Bubble (1 1)
op-hook qidSymbol (<Qids> : ~> Qid))] .
Any name can be used to define a bubble sort. It is the special attribute
id-hook Bubble (1 1)
in its constructor declaration that makes the sort Token a bubble sort. The second argument
of the id-hook special attribute indicates the minimum and maximum length of such bubbles
as lists of identifiers. Therefore, Token has only bubbles of size 1. To specify a bubble of any
length we would use the pair of values 1 and -1. The operator used in the declaration of the
bubble, in this case the operator token, is a bubble constructor that represents tokens in terms
of their quoted form. For example, the token abc123 is represented as token(’abc123).
We can define bubbles of any length, that is, non-empty sequences of Maude identifiers,
with the following declarations.
op bubble : QidList -> Bubble
[special
(id-hook Bubble (1 -1)
op-hook qidListSymbol (__ : QidList QidList ~> QidList)
op-hook qidSymbol (<Qids> : ~> Qid)
id-hook Exclude (.))] .

<!-- manual page 495 -->
18.3. TOKENS, BUBBLES, AND METAPARSING 483
In this case, the system will represent the bubble as a list of quoted identifiers under the con-
structor bubble. For example, the bubble ab cd ef is represented as bubble(’ab ’cd ’ef).
In some cases, we may want to make sure bubbles include balanced symbols. For instance,
to make sure parentheses are balanced in all bubbles we can use the following definition:
op bbubble : QidList -> Bubble
[special
(id-hook Bubble (1 -1 ‘( ‘))
op-hook qidListSymbol (__ : QidList QidList ~> QidList)
op-hook qidSymbol (<Qids> : ~> Qid)
id-hook Exclude (.))] .
Differenttypesofbubblescanbedefinedusingtheid-hookspecialattributeExclude, which
takes as parameter a list of identifiers to be excluded from the given bubble, that is, the bubble
being defined cannot contain such identifiers. In general, the syntax Exclude (I1 I2...Ik) is
used to exclude identifiers I1,I2,...,Ik inside tokens.
We can, for example, declare the sort NeTokenList with constructor neTokenList as a list
of identifiers, of any length greater or equal than one, excluding the identifier ‘->’ with the
following declarations.
op neTokenList : QidList -> NeTokenList
[special
(id-hook Bubble (1 -1)
op-hook qidListSymbol (__ : QidList QidList ~> QidList)
op-hook qidSymbol (<Qids> : ~> Qid)
id-hook Exclude ( -> ))] .
We are now ready to give the signature to parse modules such as NAT3 above. The following
moduleMINI-MAUDE-SYNTAXusestheabovedefinitionsofsortsToken, BubbleandNeTokenList
to define the syntax of a sublanguage of Maude, namely, many-sorted, unconditional, functional
modules, in which the declarations of sorts and operators have to be done one at a time, no
attributes are supported for operators, and variables must be declared on-the-fly.
fmod MINI-MAUDE-SYNTAX is
protecting QID-LIST .
sorts Token Bubble NeTokenList .
op token : Qid -> Token
[special
(id-hook Bubble (1 1)
op-hook qidSymbol (<Qids> : ~> Qid))] .
op bubble : QidList -> Bubble
[special
(id-hook Bubble (1 -1)
op-hook qidListSymbol (__ : QidList QidList ~> QidList)
op-hook qidSymbol (<Qids> : ~> Qid)
id-hook Exclude (.))] .
op neTokenList : QidList -> NeTokenList
[special
(id-hook Bubble (1 -1)
op-hook qidListSymbol (__ : QidList QidList ~> QidList)
op-hook qidSymbol (<Qids> : ~> Qid)
id-hook Exclude (->))] .

<!-- manual page 496 -->
484 CHAPTER 18. USER INTERFACES AND METALANGUAGE APPLICATIONS
sorts Decl DeclList PreModule Input Command .
subsort Decl < DeclList .
subsorts PreModule Command < Input .
--- including declaration
op including_. : Token -> Decl .
--- sort declaration
op sort_. : Token -> Decl .
--- operator declaration
op op_: ->_. : Token Token -> Decl .
op op_:_->_. : Token NeTokenList Token -> Decl .
--- equation declaration
op eq_=_. : Bubble Bubble -> Decl .
--- functional premodule
op fmod_is_endfm : Token DeclList -> PreModule .
op __ : DeclList DeclList -> DeclList [assoc gather(e E)] .
--- reduce command
op reduce_. : Bubble -> Command .
endfm
Notice how we explicitly declare operators that correspond to part of the top-level syntax
of Maude, and how we represent as terms of sort Bubble those pieces of the module—namely,
terms in equations—that can only be parsed afterwards with the user-defined syntax. Notice
also that not all terms of sort PreModule represent valid Maude modules. In particular, for a
term of sort PreModule to represent a Maude module all the bubbles must be correctly parsed
as terms in the module’s user-defined syntax. We sometimes refer to modules with bubbles as
premodules.
As an example, we can call the operation metaParse, from module META-LEVEL, with the
metarepresentation of the module MINI-MAUDE-SYNTAX and the previous module NAT3 trans-
formed into a list of quoted identifiers.
Maude> red in META-LEVEL :
metaParse(upModule(’MINI-MAUDE-SYNTAX, false),
’fmod ’NAT3 ’is
’sort ’Nat3 ’.
’op ’s_ ’: ’Nat3 ’-> ’Nat3 ’.
’op ’0 ’: ’-> ’Nat3 ’.
’eq ’s ’s ’s ’0 ’= ’0 ’.
’endfm,
’PreModule) .
We get the following term of sort ResultPair as a result:
result ResultPair:
{’fmod_is_endfm[ ’__[’sort_.[ ’__[’op_:_->_.[ ’__[’op_:_->_.[ ’NAT3 ,
’Nat3 ],
’s_ , ’Nat3 , ’Nat3 ],
’0 , ’Nat3 ],
’eq_=_.[ ’s ’s ’s ’0 , ’0 ]]],

<!-- manual page 497 -->
18.3. TOKENS, BUBBLES, AND METAPARSING 485
’PreModule}
Of course, Maude does not return these boxes. Instead, the system returns the bubbles
using their constructor form as specified in their corresponding declarations. For example,
the bubbles ’Nat3 and ’s ’s ’s ’0 are represented, respectively, as token(’Nat3) and
bubble(’s ’s ’s ’0). Maude returns them metarepresented. The result given by Maude is
therefore the following.
result ResultPair:
{’fmod_is_endfm[
’token[’’NAT3.Qid], ’__[’sort_.[’token[’’Nat3.Qid]],
’__[’op_:_->_.[’token[’’s_.Qid], ’neTokenList[’’Nat3.Qid], ’token[’’Nat3.Qid]],
’__[’op_:‘->_.[’token[’’0.Qid], ’token[’’Nat3.Qid]],
’eq_=_.[’bubble[’__[’’s.Qid, ’’s.Qid, ’’s.Qid, ’’0.Qid]], ’bubble[’’0.Qid]]]]]],
’PreModule}
The first component of the result pair is a metaterm of sort Term. To convert this term into
a term of sort FModule is now straightforward. As already mentioned, we first have to extract
from the term the module’s signature. For this, we can use an equationally defined function
op extractSignature : Term ~> FModule .
op extractSignature : Term FModule ~> FModule .
that goes along the term metarepresenting the premodule looking for sort, operator, and im-
portation declarations. A homonymous function is used to recursively handle each declara-
tion and add it to the module carried along as second argument. Notice that the operation
extractSignature is partial, because it is not well defined for metaterms of sort Term that do
not metarepresent terms of sort PreModule in MINI-MAUDE-SYNTAX.
Once we have extracted the signature of the module—expressed as a functional module with
no equations and no membership axioms—we can then build terms of sort EquationSet with
an equationally defined operation solveBubbles (also partial) that recursively replaces each
bubble in an equation with the result of calling metaParse with the already extracted signature
and with the quoted identifier form of the bubble.
op solveBubbles : Term FModule ~> FModule .
op solveBubblesAux : Term FModule ~> EquationSet .
Finally, thepartialoperationprocessModuletakesatermand, ifitmetarepresentsatermof
sort PreModule in MINI-MAUDE-SYNTAX, and, furthermore, the solveBubbles function succeeds
in parsing the bubbles in equations as terms, then it returns a term of sort FModule.
The complete specification of these operations is as follows:
fmod MINI-MAUDE is
protecting META-LEVEL .
vars T T1 T2 T3 : Term .
vars TL TL’ : TermList .
var QI : Qid .
var QIL : QidList .
var F : Qid .
var M : Module .
var I : Import .
vars IL : ImportList .
var S : Sort .
vars SS : SortSet .
var SsS : SubsortDeclSet .

<!-- manual page 498 -->
486 CHAPTER 18. USER INTERFACES AND METALANGUAGE APPLICATIONS
var OD : OpDecl .
var ODS : OpDeclSet .
var MbS : MembAxSet .
vars EqS EqS’ : EquationSet .
op processModule : Term ~> FModule .
eq processModule(T) = solveBubbles(T, extractSignature(T)) .
---- extractSignature
op extractSignature : Term ~> FModule .
op extractSignature : Term FModule ~> FModule .
eq extractSignature(’fmod_is_endfm[’token[QI], T])
= extractSignature(T,
fmod downTerm(QI, ’error) is nil sorts none . none none none none endfm) .
eq extractSignature(’__[T1, T2], M)
= extractSignature(T2, extractSignature(T1, M)) .
eq extractSignature(’sort_.[’token[T]], M) = addSort(M, downTerm(T, ’error)) .
eq extractSignature(’op_:_->_.[’token[T1], ’neTokenList[TL], ’token[T2]], M)
= addOpDecl(M,
op downTerm(T1, ’error) : downTerm(TL, nil) -> downTerm(T2, ’error) [none] .) .
eq extractSignature(’op_:‘->_.[’token[T1], ’token[T2]], M)
= addOpDecl(M, (op downTerm(T1, ’error) : nil -> downTerm(T2, ’error) [none] .)) .
eq extractSignature(’including_.[’token[T]], M)
= addImport(M, including downTerm(T, ’error) .) .
eq extractSignature(T, M) = M [owise] .
---- solveBubbles
op solveBubbles : Term FModule ~> FModule .
op solveBubblesAux : Term FModule ~> EquationSet .
eq solveBubbles(’fmod_is_endfm[’token[QI], T], M)
= addEquations(M, solveBubblesAux(T, M)) .
eq solveBubblesAux(’eq_=_.[T1, T2], M)
= (eq getTerm(processTerm(T1, M)) = getTerm(processTerm(T2, M)) [none] .) .
eq solveBubblesAux(’__[’eq_=_.[T1, T2], T3], M)
= (eq getTerm(processTerm(T1, M)) = getTerm(processTerm(T2, M)) [none] .
solveBubblesAux(T3, M)) .
ceq solveBubblesAux(’__[F[TL], T2], M)
= solveBubblesAux(T2, M)
if F =/= ’eq_=_. .
ceq solveBubblesAux(F[TL], M)
= none
if F =/= ’__ /\ F =/= ’eq_=_. .
op processTerm : Term Module ~> ResultPair .
eq processTerm(’bubble[T], M)
= metaParse(M, downTerm(T, nil), anyType) .
op addSort : FModule Sort -> FModule .
eq addSort(fmod QI is IL sorts SS . SsS ODS MbS EqS endfm, S)
= fmod QI is IL sorts SS ; S . SsS ODS MbS EqS endfm .

<!-- manual page 499 -->
18.3. TOKENS, BUBBLES, AND METAPARSING 487
op addOpDecl : FModule OpDecl -> FModule .
eq addOpDecl(fmod QI is IL sorts SS . SsS ODS MbS EqS endfm, OD)
= fmod QI is IL sorts SS . SsS (OD ODS) MbS EqS endfm .
op addImport : FModule Import -> FModule .
eq addImport(fmod QI is IL sorts SS . SsS ODS MbS EqS endfm, I)
= fmod QI is IL I sorts SS . SsS ODS MbS EqS endfm .
op addEquations : FModule EquationSet -> FModule .
eq addEquations(fmod QI is IL sorts SS . SsS ODS MbS EqS endfm, EqS’)
= fmod QI is IL sorts SS . SsS ODS MbS (EqS EqS’) endfm .
endfm
We have then the following reductions:
Maude> red in MINI-MAUDE :
extractSignature(
getTerm(metaParse(upModule(’MINI-MAUDE-SYNTAX, false),
’fmod ’NAT3 ’is
’op ’s_ ’: ’Nat3 ’-> ’Nat3 ’.
’sort ’Nat3 ’.
’op ’0 ’: ’-> ’Nat3 ’.
’eq ’s ’s ’s ’0 ’= ’0 ’.
’endfm,
’PreModule))) .
rewrites: 22 in 2ms cpu (4ms real) (7399 rewrites/second)
result FModule: fmod ’NAT3 is
nil
sorts ’Nat3 .
none
op ’0 : nil -> ’Nat3 [none] .
op ’s_ : ’Nat3 -> ’Nat3 [none] .
none
none
endfm
Maude> red in MINI-MAUDE :
processModule(
getTerm(metaParse(upModule(’MINI-MAUDE-SYNTAX, false),
’fmod ’NAT3 ’is
’sort ’Nat3 ’.
’op ’s_ ’: ’Nat3 ’-> ’Nat3 ’.
’op ’0 ’: ’-> ’Nat3 ’.
’eq ’s ’s ’s ’0 ’= ’0 ’.
’endfm,
’PreModule))) .
rewrites: 42 in 0ms cpu (1ms real) (46614 rewrites/second)
result FModule: fmod ’NAT3 is
nil
sorts ’Nat3 .
none
op ’0 : nil -> ’Nat3 [none] .
op ’s_ : ’Nat3 -> ’Nat3 [none] .
none
eq ’s_[’s_[’s_[’0.Nat3]]] = ’0.Nat3 [none] .

<!-- manual page 500 -->
488 CHAPTER 18. USER INTERFACES AND METALANGUAGE APPLICATIONS
endfm
Maude> red in MINI-MAUDE :
processModule(
getTerm(metaParse(upModule(’MINI-MAUDE-SYNTAX, false),
’fmod ’NAT3 ’is
’sort ’Nat3 ’.
’op ’s_ ’: ’Nat3 ’-> ’Nat3 ’.
’op ’0 ’: ’-> ’Nat3 ’.
’eq ’s ’s ’s ’N:Nat3 ’= ’N:Nat3 ’.
’endfm,
’PreModule))) .
result FModule: fmod ’NAT3 is
nil
sorts ’Nat3 .
none
op ’0 : nil -> ’Nat3 [none] .
op ’s_ : ’Nat3 -> ’Nat3 [none] .
none
eq ’s_[’s_[’s_[’N:Nat3]]] = ’N:Nat3 [none] .
endfm
18.4 The LOOP-MODE module
The STD-STREAM module presented in Section 9.1 has been used in the previous sections to
define interfaces. Indeed, the module STD-STREAM provides support for communication with
external objects (see Section 9), thus making it possible to develop more general and flexible
solutions for dealing with input/output. However, we may want to define something much
simpler, and do not want the overload of the required complexity. The LOOP-MODE provides a
very simple read-eval-print loop that may be enough for simple interactions and applications.
Using object-oriented concepts, the LOOP-MODE module shown below specifies a general
input/output facility, extending the module QID-LIST (see Section 8.11), into a generic read-
eval-print loop.
mod LOOP-MODE is
protecting QID-LIST .
sorts State System .
op [_,_,_] : QidList State QidList -> System [ctor special (...)] .
endm
The operator [_,_,_] can be seen as an object—that we call the loop object—with an input
stream (the first argument), an output stream (the third argument), and a state (given by its
second argument).
Since with the loop object only one input stream is supported (the current terminal), the
way to distinguish the input passed to the loop object from the input passed to the Maude
system—either modules or commands—is by enclosing them in parentheses. When something
enclosed in parentheses is written after the Maude prompt, it is converted into a list of quoted
identifiers. This is done by the system by first breaking the input stream into a sequence of
Maude identifiers (see Section 3.1) and then converting each of these identifiers into a quoted
identifier by putting a quote in front of it, and appending the results into a list of quoted
identifiers, which is then placed in the first slot of the loop object.2 The output is handled
2The tokenize function tokenizes strings into lists of quoted identifiers (see Section 8.12).

<!-- manual page 501 -->
18.4. THE LOOP-MODE MODULE 489
in the reverse way, that is, the list of quoted identifiers placed in the third slot of the loop
object is displayed on the terminal after applying the inverse process of “unquoting” each of the
identifiers in the list.3 However, the output stream is not cleared at the time when the output
is printed; it is instead cleared when the next input is entered. We can think of the input and
output events as implicit rewrites that transfer—in a slightly modified, quoted or unquoted
form—the input and output data between two objects, namely the loop object and the “user”
or “terminal” object.
Besides having input and output streams, terms of sort System give us the possibility of
maintaining a state in their second component. This state has been declared in a completely
generic way. In fact, the sort State in LOOP-MODE does not have any constructors. This gives
complete flexibility for defining the terms we want to have for representing the state of the
loop in each particular application. In this way, we can use this input/output facility not only
for building user interfaces for applications written in Maude, but also for uses of Maude as a
metalanguage, where the object language being implemented may be completely different from
Maude. For each such tool or language the nature of the state of the system may be completely
different. We can tailor the State sort to any such application by importing LOOP-MODE in a
module in which we define the state structure and the rewrite rules for changing the state and
interacting with the loop.
To illustrate the use of the LOOP-MODE module, let us re-implement the interface in Sec-
tion 18.1 using it instead of the facilities provided by the STD-STREAM module as is done there.
Please, compare the specification below with the one in Section 18.1, because they are very
similar. We will focus on the differences.
omod VENDING-MACHINE-LOOP-MODE is
including LOOP-MODE .
including VENDING-MACHINE-GRAMMAR .
protecting BUYING-STRATS .
protecting CONVERSION .
protecting LEXICAL .
As with the STD-STREAM module, let us keep the state of the loop in an object of class VM.
class VM | action : Action, marking : Marking .
subsort Object < State .
Then, we define constants VM-GRAMMAR, idle, vm and vending machine as in Section 18.1.
However, note that vending machine defines now the initial state of the system as a loop
object. The vm object is as above, but notice that we create the loop object with an empty
input stream (nil) and the welcome message in its output stream. As we will see below, when
we initiate the loop with the loop command and this constant, the loop object will take the list
of quoted identifiers in its output stream and will print the corresponding text in the terminal.
op VM-GRAMMAR : -> Module .
eq VM-GRAMMAR = upModule(’VENDING-MACHINE-GRAMMAR, false) .
op idle : -> Action .
op vm : -> Oid .
op vending machine : -> System .
eq vending machine
= [nil, < vm : VM | marking : null, action : idle >, ’Vending ’machine] .
We need some variables for the definition of the rules defining the interaction. Note the
definition of variable NQIL of type NeQidList (Section 8.14.1). Whilst with STD-STREAM the
3The printTokens function takes a list of quoted identifiers and converts it into a string with the usual
printing conventions (see Section 8.12).

<!-- manual page 502 -->
490 CHAPTER 18. USER INTERFACES AND METALANGUAGE APPLICATIONS
text comes as strings, with LOOP-MODE it comes as lists of quoted identifiers. The fact that the
lists matching the variable will be non-empty simplifies the interaction.
var O : Oid .
var NQIL : NeQidList .
var A : Action .
var I : Item .
var C : Coin .
var M : Marking .
var N : Nat .
Our first rule takes a list of quoted identifiers from the input stream, parses it and passes
it to the vm object in the state of the loop object. If there is a ’quit in the input channel,
the loop object is terminated, with a ’goodbye in its output channel. If there is non-parsable
input, it is discarded and a message is placed in the output channel. Note that both inputs
and outputs are represented as lists of quoted identifiers. Inputs are tokenized, and outputs are
pretty printed correspondingly.
rl [NQIL, < vm : VM | action : idle, marking : M >, nil]
=> if NQIL == ’quit
then [nil, none, ’goodbye]
else if metaParse(VM-GRAMMAR, NQIL, ’Action) :: ResultPair
then [nil, < vm : VM |
action : downTerm(
getTerm(
metaParse(VM-GRAMMAR, NQIL, ’Action)),
idle),
marking : M >, nil]
else [NQIL, < vm : VM | action : idle, marking : M >, ’Invalid ’input]
fi
fi .
Auxiliary functions showBasket and showCredit are the same ones used in Section 18.1.
op showBasket : Marking -> QidList .
eq showBasket(I M)
= metaPrettyPrint(upModule(’VENDING-MACHINE-SIGNATURE, false),
upTerm(I))
showBasket(M) .
eq showBasket(C M) = showBasket(M) .
eq showBasket(null) = nil .
op showCredit : Marking -> QidList .
eq showCredit(C M)
= metaPrettyPrint(upModule(’VENDING-MACHINE-SIGNATURE, false),
upTerm(C))
showCredit(M) .
eq showCredit(I M) = showCredit(M) .
eq showCredit(null) = nil .
The rest of the rules specify the action the way in which the vm object reacts to each of the
commands.
rl [nil, < vm : VM | action : show basket, marking : M >, nil]
=> [nil, < vm : VM | action : idle, marking : M >, ’basket: showBasket(M)] .
rl [nil, < vm : VM | action : show credit, marking : M >, nil]

<!-- manual page 503 -->
18.4. THE LOOP-MODE MODULE 491
=> [nil, < vm : VM | action : idle, marking : M >, ’basket: showCredit(M)] .
rl [nil, < vm : VM | action : insert q, marking : M >, nil]
=> [nil,
< vm : VM | action : idle,
’one ’quarter ’introduced] .
marking : downTerm(insertCoin(’add-q, upTerm(M)), null) >,
rl [nil, < vm : VM | action : insert $, marking : M >, nil]
=> [nil,
< vm : VM | action : idle,
’one ’dollar ’introduced] .
marking : downTerm(insertCoin(’add-$, upTerm(M)), null) >,
rl [nil, < vm : VM | action : (buy N c (s)), marking : M >, nil]
=> [nil,
< vm : VM | action : idle,
marking : downTerm(onlyNitems(upTerm(M), ’buy-c, N), null) >,
qid(string(N, 10)) ’cakes ’bought] .
rl [nil, < vm : VM | action : (buy N a (s)), marking : M >, nil]
=> [nil,
< vm : VM | action : idle,
marking : downTerm(onlyNitems(upTerm(M), ’buy-a, N), null) >,
qid(string(N, 10)) ’apples ’bought] .
endom
The same interaction presented in Section 18.1 now looks as follows.
Maude> loop vending machine .
Vending machine
Maude> (insert $)
one dollar introduced
Maude> (show credit)
basket: $
Maude> (insert $)
one dollar introduced
Maude> (insert q)
one quarter introduced
Maude> (buy 1 apple(s))
Invalid input
Maude> (buy 1 a(s))
1 apples bought
Maude> (show basket)
basket: a
Maude> (show credit)
basket: $ q q
Maude> (insert $)
one dollar introduced
Maude> (buy 3 a(s))
3 apples bought
Maude> (show basket)
basket: a a a a
Maude> (show credit)
basket: q

<!-- manual page 504 -->
492 CHAPTER 18. USER INTERFACES AND METALANGUAGE APPLICATIONS
Maude> (quit)
goodbye
Maude>
Note that the loop command initiates the loop, placing inputs in parentheses in its input
channel and taking lists in its output channel and printing them in the terminal. Thus, even
though the loop is active, inputs without parentheses will be handled by Maude as any other
input. Let us illustrate this idea with the following interaction with the Maude interpreter.
Maude> loop vending machine .
Vending machine
For example, we can see the state of the loop with the continue command (Appendix A.2).
Maude> cont .
rewrites: 0 in 0ms cpu (0ms real) (0 rewrites/second)
result System: [nil, < vm : VM | action : idle, marking : null >, ’Vending ’machine]
Whilst the loop is not broken we can enter inputs for it, e.g.,
Maude> (insert $)
one dollar introduced
We can also activate the trace to see what is going on (please, note the ellipsis).
Maude> set trace on .
Maude> (show credit)
*********** rule
rl [NQIL, < vm : V:VM | Atts:AttributeSet, action : idle, marking : M >, nil] => if
NQIL == ’quit then [nil, none, ’goodbye] else if metaParse(VM-GRAMMAR, NQIL,
’Action) :: ResultPair then [nil, < vm : V:VM | Atts:AttributeSet, action :
downTerm(getTerm(metaParse(VM-GRAMMAR, NQIL, ’Action)), idle), marking : M >, nil]
else [NQIL, < vm : V:VM | Atts:AttributeSet, action : idle, marking : M >, ’Invalid
’input] fi fi .
NQIL --> ’show ’credit
V:VM --> VM
Atts:AttributeSet --> (none).AttributeSet
M --> $
[’show ’credit, < vm : VM | action : idle, marking : $ >, nil]
--->
if ’show ’credit == ’quit then [nil, none, ’goodbye] else if metaParse(VM-GRAMMAR,
’show ’credit, ’Action) :: ResultPair then [nil, < vm : VM | none, action :
downTerm(getTerm(metaParse(VM-GRAMMAR, ’show ’credit, ’Action)), idle), marking
: $ >, nil] else [’show ’credit, < vm : VM | none, action : idle, marking : $ >,
’Invalid ’input] fi fi
*********** equation
...
basket: $
If one forgets the parentheses, Maude may interpret the inputs not as intended.
Maude> quit
Bye.

