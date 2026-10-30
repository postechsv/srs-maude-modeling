# Chapter 23: Maude Grammar (manual pages 587-596)

<!-- manual page 587 -->
Appendix B
Maude Grammar
This chapter describes the syntax of Maude using the following extended BNF notation: the
symbols ‘(’ and ‘)’ are used as metaparentheses; the symbol ‘|’ is used to separate alternatives;
square bracket pairs, ‘[’ and ‘]’, enclose optional syntax; ‘*’ indicates zero or more repetitions
of preceding unit; ‘+’ indicates one or more repetitions of preceding unit; and the string “x”
denotes x literally. As an application of this notation, A(, A)* indicates a non-empty list of A’s
separated by commas. Finally, %%% indicates comments in the syntactic description, as opposed
to comments in the Maude code.
B.1 The grammar
⟨MaudeTop⟩ ::=
( ⟨SystemCommand⟩ | ⟨Command⟩ | ⟨DebuggerCommand⟩ |
⟨Module⟩ | ⟨Theory⟩ | ⟨View⟩ )+
⟨SystemCommand⟩ ::= in ⟨FileName⟩ | load ⟨FileName⟩ | sload ⟨FileName⟩ |
quit | eof | popd | pwd |
cd ⟨Directory⟩ | push ⟨Directory⟩ |
ls [ ⟨LsFlag⟩ ] [ ⟨Directory⟩ ]
⟨Command⟩ ::= select ⟨ModId⟩. |
parse [ in ⟨ModId⟩ : ] ⟨Term⟩. |
[ debug ] reduce [ in ⟨ModId⟩ : ] ⟨Term⟩. |
[ debug ] rewrite [ [ ⟨Nat⟩ ] ] [ in ⟨ModId⟩ : ] ⟨Term⟩. |
[ debug ] frewrite [ ⟨DoubleBound⟩ ] [ in ⟨ModId⟩ :
] ⟨Term⟩. |
[ debug ] erewrite [ ⟨DoubleBound⟩ ] [ in ⟨ModId⟩ : ]
⟨Term⟩. |
[ debug ] ( srewrite | dsrewrite ) [ [ ⟨Nat⟩ ] ] [ in ⟨ModId⟩ : ]
⟨Term⟩ using ⟨Strat⟩. |
check [ in ⟨ModId⟩ : ] ⟨Term⟩. |
( match | xmatch ) [ [ ⟨Nat⟩ ] ] [ in ⟨ModId⟩ : ]
⟨Term⟩ <=? ⟨Term⟩ [ such that ⟨Condition⟩ ]. |
[ debug ] variant match [ [ ⟨Nat⟩ ] ] [ in ⟨ModId⟩ : ]
⟨Term⟩ <=? ⟨Term⟩ [ such that ⟨Condition⟩ ]. |
575

<!-- manual page 588 -->
576 APPENDIX B. MAUDE GRAMMAR
[ irredudant ] unify [ [ ⟨Nat⟩ ] ] [ in ⟨ModId⟩ : ]
⟨UnificationEquation⟩ ( /\ ⟨UnificationEquation⟩ )*
. |
[ debug ] [ filtered ] variant unify [ [ ⟨Nat⟩ ] ] [ in ⟨ModId⟩ : ]
⟨UnificationEquation⟩ ( /\ ⟨UnificationEquation⟩ )*
. |
[ debug ] get [ irredundant ] variants [ [ ⟨Nat⟩ ] ] [ in ⟨ModId⟩ : ] ⟨Term⟩. |
[ debug ] search [ ⟨DoubleBound⟩ ] [ in ⟨ModId⟩ : ] ⟨Term⟩ ⟨SearchType⟩ ⟨Term⟩
[ such that ⟨Condition⟩ ]. |
[ debug ] [ ⟨NarrowingOptionList⟩ ] vu-narrow [ ⟨VariantOptionList⟩ ] [ ⟨DoubleBound⟩ ]
[ in ⟨ModId⟩ : ] ⟨Term⟩ ⟨SearchType⟩ ⟨Term⟩. |
[ debug ] continue ⟨Nat⟩. |
loop [ in ⟨ModId⟩ : ] ⟨Term⟩. |
( ⟨TokenString⟩ ) |
trace ( select | deselect | include | exclude )
( ⟨OpId⟩ | ( ⟨OpForm⟩ ) )+. |
print ( conceal | reveal ) ( ⟨OpId⟩ | ( ⟨OpForm⟩ ) )+. |
break ( select | deselect ) ( ⟨OpId⟩ | ( ⟨OpForm⟩ ) )+. |
show ⟨ShowItem⟩ [ ⟨ModId⟩ ]. |
show [ processed ] view [ ⟨ViewId⟩ ]. |
show modules . |
show views . |
show search graph . |
show path [ labels | states ] ⟨Nat⟩. |
show frontier states . |
show most general states . |
do clear memo [ ⟨ModId⟩ ]. |
set ⟨SetOption⟩ ( on | off ).
⟨DoubleBound⟩ ::= [ ⟨Nat⟩ [, ⟨Nat⟩ ] ] | [ , ⟨Nat⟩ ]
⟨SearchType⟩ ::= =>1 | =>+ | =>* | =>! | =>#
⟨UnificationEquation⟩ ::= ⟨Term⟩ =? ⟨Term⟩
⟨ShowItem⟩ ::= module | all | desugared | sorts | ops | vars | mbs |
eqs | rls | strats | sds | summary | kinds | profile
⟨SetOption⟩ ::= show ⟨ShowOption⟩ |
print ⟨PrintOption⟩ |
trace [ ⟨TraceOption⟩ ] |
break | verbose | profile |
clear ( memo | rules | profile ) |
protect ⟨ModId⟩ |
extend ⟨ModId⟩ |
generate-by ⟨ModId⟩ |
include ⟨ModId⟩ |
oo include ⟨ModID⟩
⟨ShowOption⟩ ::= advise | stats | loop stats | timing |
loop timing | breakdown | command | gc

<!-- manual page 589 -->
B.1. THE GRAMMAR 577
⟨PrintOption⟩ ::= mixfix | flat | with parentheses |
with aliases | conceal | number | rat | color |
format | graph | attribute | attribute newline |
constants with sorts | latex
⟨TraceOption⟩ ::= condition | whole | substitution | select |
mbs | eqs | rls | sds | rewrite | body
⟨VariantOptionList⟩ ::= { ⟨VariantOption⟩ (, ⟨VariantOption⟩ )* }
⟨VariantOption⟩ ::= filter | delay
⟨NarrowingOptionList⟩ ::= { ⟨NarrowingOption⟩ (, ⟨NarrowingOption⟩ )* }
⟨NarrowingOption⟩ ::= fold | vfold | path
⟨DebuggerCommand⟩ ::= resume . | abort . | step . | where .
⟨Module⟩ ::= fmod ⟨ModId⟩ [ ⟨ParameterList⟩ ] is ⟨ModElt⟩* endfm |
mod ⟨ModId⟩ [ ⟨ParameterList⟩ ] is ⟨ModElt’⟩* endm |
omod ⟨ModId⟩ [ ⟨ParameterList⟩ ] is ⟨OmodElt⟩* endom |
smod ⟨ModId⟩ [ ⟨ParameterList⟩ ] is ⟨SmodElt⟩* endsm
⟨Theory⟩ ::= fth ⟨ModId⟩ is ⟨ModElt⟩* endfth |
th ⟨ModId⟩ is ⟨ModElt’⟩* endth |
oth ⟨ModId⟩ is ⟨OmodElt⟩* endoth |
sth ⟨ModId⟩ is ⟨SmodElt⟩* endsth
⟨View⟩ ::= view ⟨ViewId⟩ [ ⟨ParameterList⟩ ] from ⟨ModExp⟩ to ⟨ModExp⟩ is
⟨ViewElt⟩*
endv
⟨ParameterList⟩ ::= { ⟨ParameterDecl⟩ (, ⟨ParameterDecl⟩ )* }
⟨ParameterDecl⟩ ::= ⟨ParameterId⟩ :: ⟨ModExp⟩
⟨ModElt⟩ ::= including ⟨ModExp⟩. |
extending ⟨ModExp⟩. |
protecting ⟨ModExp⟩. |
generated-by ⟨ModExp⟩. |
sorts ⟨Sort⟩+. |
subsorts ⟨Sort⟩+ ( < ⟨Sort⟩+ )+. |
op ⟨OpForm⟩ : ⟨Type⟩* ⟨Arrow⟩ ⟨Type⟩ [ ⟨Attr⟩ ]. |
ops ( ⟨OpId⟩ | ( ⟨OpForm⟩ ) )+ : ⟨Type⟩* ⟨Arrow⟩ ⟨Type⟩ [ ⟨Attr⟩ ]. |
vars ⟨VarId⟩+ : ⟨Type⟩. |
⟨Statement⟩ [ ⟨StatementAttr⟩ ].
⟨ViewElt⟩ ::= var ⟨VarId⟩+ : ⟨Type⟩. |

<!-- manual page 590 -->
578 APPENDIX B. MAUDE GRAMMAR
sort ⟨Sort⟩ to ⟨Sort⟩. |
label ⟨LabelId⟩ to ⟨LabelId⟩. |
op ⟨OpForm⟩ to ⟨OpForm⟩. |
op ⟨OpForm⟩ : ⟨Type⟩* ⟨Arrow⟩ ⟨Type⟩ to ⟨OpForm⟩. |
op ⟨Term⟩ to term ⟨Term⟩. |
class ⟨Class⟩ to ⟨Class⟩. |
attr ⟨AttributeId⟩ [ : ⟨Type⟩ ] to ⟨AttributeId⟩. |
msg ⟨OpForm⟩ to ⟨OpForm⟩. |
msg ⟨OpForm⟩ : ⟨Type⟩* ⟨Arrow⟩ ⟨Type⟩ to ⟨OpForm⟩. |
msg ⟨Term⟩ to term ⟨Term⟩. |
strat ⟨StratId⟩ to ⟨StratId⟩. |
strat ⟨StratId⟩ [ : ⟨Type⟩* ] @ ⟨Type⟩ to ⟨StratId⟩. |
strat ⟨StratCall⟩ to expr ⟨Strat⟩.
⟨ModExp⟩ ::= ⟨ModId⟩ |
( ⟨ModExp⟩ ) |
⟨ModExp⟩ + ⟨ModExp⟩ |
⟨ModExp⟩ * ⟨Renaming⟩
⟨ModExp⟩ { ⟨ViewId⟩ (, ⟨ViewId⟩ )* }
⟨Renaming⟩ ::= ( ⟨RenamingItem⟩ (, ⟨RenamingItem⟩ )* )
⟨RenamingItem⟩ ::= sort ⟨Sort⟩ to ⟨Sort⟩ |
label ⟨LabelId⟩ to ⟨LabelId⟩ |
op ⟨OpForm⟩ ⟨ToPartRenamingItem⟩ |
op ⟨OpForm⟩ : ⟨Type⟩* ⟨Arrow⟩ ⟨Type⟩ ⟨ToPartRenamingItem⟩
class ⟨Class⟩ to ⟨Class⟩ |
attr ⟨AttributeId⟩ [ : ⟨Type⟩ ] to ⟨AttributeId⟩ |
msg ⟨OpForm⟩ ⟨ToPartRenamingItem⟩ |
msg ⟨OpForm⟩ : ⟨Type⟩* ⟨Arrow⟩ ⟨Type⟩ ⟨ToPartRenamingItem⟩
strat ⟨StratId⟩ to ⟨StratId⟩ |
strat ⟨StratId⟩ [ : ⟨Type⟩* ] @ ⟨Type⟩ to ⟨StratId⟩
⟨ToPartRenamingItem⟩ ::= to ⟨OpForm⟩ [ ⟨Attr⟩ ]
⟨Arrow⟩ ::= -> | ~>
⟨Type⟩ ::= ⟨Sort⟩ | ⟨Kind⟩
⟨Kind⟩ ::= [ ⟨Sort⟩ (, ⟨Sort⟩ )* ]
⟨Sort⟩ ::= ⟨SortId⟩ | ⟨Sort⟩ { ⟨Sort⟩ (, ⟨Sort⟩ )* }
⟨Class⟩ ::= ⟨Sort⟩ %%% without underscores
⟨ModElt’⟩ ::= ⟨ModElt⟩ |
⟨Statement’⟩ [ ⟨StatementAttr⟩ ].
⟨OmodElt⟩ ::= ⟨ModElt’⟩ |

<!-- manual page 591 -->
B.1. THE GRAMMAR 579
class ⟨Class⟩ [ | ⟨ClassDef⟩ ]. |
subclass ⟨Class⟩+ ( < ⟨Class⟩+ )+. |
msg ⟨OpForm⟩ : ⟨Type⟩* ⟨Arrow⟩ ⟨Type⟩ [ ⟨Attr⟩ ]. |
msgs ( ⟨OpId⟩ | ( ⟨OpForm⟩ ) )+ : ⟨Type⟩* ⟨Arrow⟩ ⟨Type⟩ [ ⟨Attr⟩ ].
⟨SmodElt⟩ ::= ⟨ModElt’⟩ |
strats ⟨StratId⟩+ [ : ⟨Type⟩* ] @ ⟨Type⟩ [ ⟨StratAttr⟩ ].
⟨StratStatement⟩ [ ⟨StatementAttr⟩ ].
⟨Statement⟩ ::= mb [ ⟨Label⟩ ] ⟨Term⟩ : ⟨Sort⟩ |
cmb [ ⟨Label⟩ ] ⟨Term⟩ : ⟨Sort⟩ if ⟨Condition⟩ |
eq [ ⟨Label⟩ ] ⟨Term⟩= ⟨Term⟩ |
ceq [ ⟨Label⟩ ] ⟨Term⟩= ⟨Term⟩ if ⟨Condition⟩
⟨Statement’⟩ ::= rl [ ⟨Label⟩ ] ⟨Term⟩ => ⟨Term⟩ |
crl [ ⟨Label⟩ ] ⟨Term⟩ => ⟨Term⟩ if ⟨Condition’⟩
⟨StratStatement⟩ ::= sd ⟨StratCall⟩ := ⟨Strat⟩ |
csd ⟨StratCall⟩ := ⟨Strat⟩ if ⟨Condition⟩
⟨ClassDef⟩ ::= ⟨AttributeDecl⟩ (, ⟨AttributeDecl⟩ )*
⟨AttributeDecl⟩ ::= ⟨AttributeId⟩ [ : ] ⟨Type⟩
⟨Label⟩ ::= [ ⟨LabelId⟩ ] :
⟨Condition⟩ ::= ⟨ConditionFragment⟩ ( /\ ⟨ConditionFragment⟩ )*
⟨Condition’⟩ ::= ⟨ConditionFragment’⟩
( /\ ⟨ConditionFragment’⟩ )*
⟨ConditionFragment⟩ ::= ⟨Term⟩= ⟨Term⟩ | ⟨Term⟩ := ⟨Term⟩
| ⟨Term⟩ : ⟨Sort⟩
⟨ConditionFragment’⟩ ::= ⟨ConditionFragment⟩ | ⟨Term⟩ => ⟨Term⟩
⟨Attr⟩ ::=
[ ( assoc | comm |
[ left | right ] id: ⟨Term⟩ |
idem | iter | memo | ditto |
config | obj | msg | pcons | ctor | portal |
metadata ⟨StringId⟩
strat ( ⟨Nat⟩+ ) |
poly ( ⟨Nat⟩+ ) |
frozen [ ( ⟨Nat⟩+ ) ] |
prec ⟨Nat⟩ |
gather ( ( e | E | & )+ ) |
format ( ⟨Token⟩+ ) |
special ( ⟨Hook⟩+ ) )+ ] |

<!-- manual page 592 -->
580 APPENDIX B. MAUDE GRAMMAR
latex ( ⟨LatexFragment⟩ )
⟨StratAttr⟩ ::= [ metadata ⟨StringId⟩ ]
⟨StatementAttr⟩ ::=
[ ( nonexec | dnt |
metadata ⟨StringId⟩ |
label ⟨LabelId⟩ |
print ⟨PrintItem⟩* )+ ]
⟨StatementAttrEq⟩ ::=
[ ( ⟨StatementAttr⟩ | otherwise | variant )+ ]
⟨StatementAttrRl⟩ ::=
[ ( ⟨StatementAttr⟩ | narrowing )+ ]
⟨PrintItem⟩ ::= ⟨StringId⟩ | ⟨VarId⟩ | ⟨VarAndSortId⟩
⟨Hook⟩ ::= id-hook ⟨Token⟩ [ ( ⟨TokenString⟩ ) ] |
( op-hook | term-hook ) ⟨Token⟩ ( ⟨TokenString⟩ )
⟨Strat⟩ ::= idle | fail |
⟨RuleApp⟩ | top( ⟨RuleApp⟩ ) |
⟨Strat⟩ ? ⟨Strat⟩ : ⟨Strat⟩ |
⟨TestVariant⟩ ⟨Term⟩ [ such that ⟨Condition⟩ ]
⟨Strat⟩ ; ⟨Strat⟩ |
⟨Strat⟩ | ⟨Strat⟩ |
⟨Strat⟩ * |
⟨MrewVariant⟩ ⟨Term⟩ [ such that ⟨Condition⟩ ] by ⟨VarStratList⟩
⟨StratCall⟩
⟨Strat⟩ +
⟨Strat⟩ or-else ⟨Strat⟩
not( ⟨Strat⟩ )
⟨Strat⟩ !
try( ⟨Strat⟩ )
test( ⟨Strat⟩ )
⟨RuleApp⟩ ::= ⟨LabelId⟩ [ [ ⟨Substitution⟩ ] ] [ { ⟨Strat⟩ (, ⟨Strat⟩)* } ]
⟨Substitution⟩ ::= ⟨VarId⟩ <- ⟨Term⟩ |
⟨Substitution⟩, ⟨Substitution⟩
⟨StratCall⟩ ::= ⟨StratId⟩ [ () ] |
⟨StratId⟩ ( ⟨Term⟩ (, ⟨Term⟩)* )
⟨VarStratList⟩ ::= ⟨VarId⟩ using ⟨Strat⟩ |
⟨VarStratList⟩, ⟨VarStratList⟩
⟨TestVariant⟩ ::= match | xmatch | amatch

<!-- manual page 593 -->
B.2. SYNONYMS 581
⟨MrewVariant⟩ ::= matchrew | xmatchrew | amatchrew
⟨FileName⟩ %%% OS dependent
⟨Directory⟩ %%% OS dependent
⟨LsFlag⟩ %%% OS dependent
⟨StringId⟩ %%% characters enclosed in double quotes "..."
⟨ModId⟩ %%% simple identifier, by convention all capitals, with hyphens to separate words
⟨ViewId⟩ %%% simple identifier, by convention capitalized
⟨ParameterId⟩ %%% simple identifier, by convention single capital
⟨SortId⟩ %%% simple identifier, by convention capitalized. It must
not include parentheses, brackets, braces, commas, periods,
or colons, and <, ->, ~>, and @ are excluded.
⟨VarId⟩ %%% simple identifier, except :, by convention capitalized
⟨VarAndSortId⟩ %%% an identifier consisting of a variable name
followed by a colon followed by a sort name
⟨OpId⟩ %%% identifier possibly with underscores
⟨OpForm⟩ ::= ⟨OpId⟩ | ( ⟨OpForm⟩ ) | ⟨OpForm⟩+
⟨Nat⟩ %%% a natural number
⟨Term⟩ ::= ⟨Token⟩ | ( ⟨Term⟩ ) | ⟨Term⟩+
⟨Token⟩ %%% sequence of printable ASCII characters delimited by
whitespace. The symbols (, ), [, ], {, } and comma form
separate tokens themselves, unless backquoted
⟨TokenString⟩ ::= ⟨Token⟩ | ( ⟨TokenString⟩ ) | ⟨TokenString⟩*
⟨LabelId⟩ %%% simple identifier
⟨StratId⟩ %%% simple identifier, except @ and :
⟨AttributeId⟩ %%% simple identifier, except [, ], {, } and comma
In parsing module expressions, instantiation has higher precedence than renaming, which
in turn has higher precedence than summation.
B.2 Synonyms
sort = sorts
subsort= subsorts
var = vars
Command only synonyms:
advise= advisory= advisories
alias= aliases
attr = attribute
cmd= command
cond= condition
cont = continue
dsrew= dsrewrite
eqs= eq
erew = erewrite

<!-- manual page 594 -->
582 APPENDIX B. MAUDE GRAMMAR
flat= flattened
frew= frewrite
fvu-narrow= {fold} vu-narrow
irred= irreducible
kinds= components
label= labels
mbs= mb
nar = narrow
paren= parens= parentheses
q= quit
rat = rational
red= reduce
rew = rewrite
rls= rl= rule= rules
s.t. = such that
srew = srewrite
subst= substitution
sds= sd
state = states
Module only synonyms:
assoc = associative
ceq= cq
class= classes
comm = commutative
config= configuration
ctor = constructor
ex = extending
gb= generated-by
id:= identity:
idem= idempotent
inc= including
iter= iterated
msg= message
obj= object
owise= otherwise
poly= polymorphic
prec= precedence
pr= protecting
subclass= subclasses
strat = strategy strats = strat %%% reduction strategy attribute
%%% rewriting strategy declaration
B.3 Lexical and other Issues
Tokens are sequences of printable ASCII characters delimited by white space, except that ‘(’,
‘)’, ‘[’, ‘]’, ‘{’, ‘}’, and ‘,’ are always considered as single character tokens, unless backquoted.
Single line comments are started by one of ***or–-, and ended by the end of line. Multiline
comments are started by ***( and ended by ). Parentheses (whether backquoted or not) must

<!-- manual page 595 -->
B.3. LEXICAL AND OTHER ISSUES 583
balance within multiline comments.
String identifiers use C backslash conventions [77, Section A2.5.2].
Operators having mixfix syntax with mismatched parentheses generate a warning and the
mixfix syntax is disabled. Although not recommended, it is possible declaring such operators
using backquote syntax.
fmod FOO is
sort Foo .
op ‘(_‘)‘) : Foo -> Foo .
op ‘)_‘( : Foo -> Foo .
op _+_ : Foo Foo -> Foo .
op a : -> Foo .
endfm
When such declarations are used, Maude gives warning messages. For instance, loading the
previous FOO module produces the following warnings:
Warning: <standard input>, line 3 (fmod FOO): mismatched parentheses in
operator ‘(_‘)‘). It will be treated as having prefix syntax only.
Warning: <standard input>, line 4 (fmod FOO): mismatched parentheses in
operator ‘)_‘(. It will be treated as having prefix syntax only.
Then, as advised, terms using these operators will be printed using mixfix syntax. For
example,
Maude> red ‘(_‘)‘)(a) + a .
reduce in FOO : ‘(_‘)‘)(a) + a .
rewrites: 0 in 0ms cpu (0ms real) (0 rewrites/second)
result Foo: ‘(_‘)‘)(a) + a
Maude> red ‘)_‘((a + ‘(_‘)‘)(a)) .
reduce in FOO : ‘)_‘((a + ‘(_‘)‘)(a)) .
rewrites: 0 in 0ms cpu (0ms real) (0 rewrites/second)
result Foo: ‘)_‘((a + ‘(_‘)‘)(a))
To avoid inadvertently running out of memory, the limit for integer exponents and left shifts
is ULONG_MAX (9,223,372,036,854,775,807 on 64-bit hardware, 4,294,967,295 on 32-bit
hardware) which is a hard limit imposed by GMP. However, if the first argument is 0, 1 or -1
for exponentiation, or 0 for left shift, then the answer is trivial and the second arguments can
be arbitrarily long.

<!-- manual page 596 -->
584 APPENDIX B. MAUDE GRAMMAR

