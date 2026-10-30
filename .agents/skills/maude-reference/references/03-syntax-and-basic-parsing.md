# Chapter 3: Syntax and Basic Parsing (manual pages 37-54)

<!-- manual page 37 -->
Chapter 3
Syntax and Basic Parsing
This chapter introduces the basic syntactic ingredients of all Maude specifications: identifiers,
module names, sort names, and operator declarations. Other syntactic parts of Maude specifi-
cations, like equations and rules, will appear in the following chapters.
Some syntax is presented in an informal way by means of general schemes; a formal BNF
grammar of the language can be found in Chapter B.
The chapter finishes explaining some features that can be used to reduce parsing ambiguities
in the user-definable syntax, including mixfix operator declarations, supported by Maude.
3.1 Identifiers
In Maude, identifiers are the basic syntactic elements, used to name modules, sorts, classes,
and to form operator, objects’ attributes and strategy names. For example, NAT, Nat, and
hello-world are identifiers. In general, an identifier in Maude is any finite sequence of ASCII
characters such that:
• It does not contain any white space. For example, the sequence ‘abc def’ is not one
identifier, but two.
• The characters ‘{’, ‘}’, ‘(’, ‘)’, ‘[’, ‘]’ and ‘,’ are special, in that they break a sequence of
characters into several identifiers. For example, the sequence ab{c,d}ef counts as seven
identifiers, namely, ab, {, c, ,, d, }, and ef.
• The backquote character ‘‘’ is used as an escape character to indicate that a blank
space or the special characters do not break the sequence. Consequently, backquotes
can only appear immediately before any of the special characters, or between two non-
empty strings of characters—with neither the ending of the first string nor the beginning
of the second string being another backquote—for exactly these purposes. For exam-
ple, 1‘ab‘{c‘,d‘}ef is a single identifier. Maude’s pretty printer will display such an
identifier in the form 1 ab{c,d}ef.
Nonprinting characters in strings use C backslash conventions [77, Section A2.5.2]. As we will
see in Sections 3.3, 6, and 7.4.2, specific restrictions apply to identifiers of sorts, class names,
and view names. Also, as we will see in Section 3.4, operator names may consist of several
identifiers, due to the presence of blanks or other special characters.
25

<!-- manual page 38 -->
26 CHAPTER 3. SYNTAX AND BASIC PARSING
3.2 Modules
In Maude the basic units of specification and programming are called modules.
1 A module
consists of syntax declarations, providing appropriate language to describe the system at hand,
and of statements, asserting the properties of such a system. The syntax declaration part is
called a signature and consists of declarations for:
• sorts, giving names for the types of data,
• subsorts, organizing the data types in a hierarchy,
• kinds, that are implicit and intuitively correspond to “error supertypes” that, in addition
to normal data, can contain “error expressions,” and
• operators, providing names for the operations that will act upon the data and allowing
us to build expressions (or terms) referring to such data.
We use symbols Σ,Σ′, etc. to denote signatures.
In Maude there are two kinds of modules: functional modules and system modules. Signa-
tures are common for both of them. The difference between functional and system modules
resides in the statements they can have:
• functional modules admit equations, identifying data, and memberships, stating typing
information for some data, while
• system modules also admit rules, describing transitions between states, in addition to
equations and memberships.
We use E,E′, etc. to denote sets of equations and memberships, and R,R′, etc. to denote sets
of rules.
From a programming point of view, a functional module is an equational-style functional
program with user-definable syntax in which a number of sorts, their elements, and functions
on those sorts are defined. From a specification viewpoint, a functional module is an equational
theory (Σ,E) with initial algebra semantics. Functional modules are described in detail in
Chapter 4, here we just discuss some of their top-level syntax. Each functional module has a
name, which is a Maude identifier. Any Maude identifier can be used, but the preferred style for
module names is an all capitalized identifier, and in the case of a compound name the different
parts are linked with hyphens. For example, a module defining numbers and operations on
them can be called NUMBERS. The top-level syntax will then be
fmod NUMBERS is
...
endfm
with ‘...’ corresponding to all the declarations of submodule importations, sorts, subsorts,
operators, variables, equations, and so on.
From a programming point of view, a system module is a declarative-style concurrent
program with user-definable syntax. From a specification viewpoint, it is a rewrite theory
(Σ,E,ϕ,R) (where ϕ specifies the frozen arguments of operators in Σ; see Section 4.4.9) with
initial model semantics. Again, each system module has a name, which is a Maude identi-
fier (a simple or single-token identifier). And as for functional modules, the preferred style is
an all capitalized name, with consecutive parts linked with hyphens in the case of compound
names. For example, a module specifying the behavior of a vending machine may be called
VENDING-MACHINE. It will then be introduced with the following top-level syntax:
mod VENDING-MACHINE is
...
endm
1As explained in Section 7.4.1, specifications can also be given in theories, with a syntax entirely similar to
that of modules, but theories, unlike modules, need not be executable.

<!-- manual page 39 -->
3.3. SORTS AND SUBSORTS 27
where again ‘...’ corresponds to all the declarations of submodule importations, sorts, subsorts,
operators, variables, equations, rules, and so on. System modules are described in detail in
Chapter 5.
In the rest of the chapter we will describe the ingredients of signatures, that is, the syn-
tactic elements common to both functional and system modules, such as sorts, subsorts, kinds,
operators, variables, and the terms that can be built on a signature, postponing the discussion
about the syntax specific to functional and system modules to Chapters 4 and 5, respectively.
3.3 Sorts and subsorts
The first thing a specification needs to declare are the types (that in the algebraic specification
community are usually called sorts) of the data being defined and the corresponding operations.
Sorts can be partially ordered via a subsort relation.
A sort is declared using the sort keyword followed by an identifier (the sort name), followed
by white space and a period, as follows:
sort ⟨Sort⟩.
and multiple sorts may be declared using the sorts keyword, as follows:
sorts ⟨Sort-1⟩... ⟨Sort-k⟩.
The period at the end of the sort declaration, as for the other types of declarations, is
crucial. Note that if either the period is missing or no space is left before and after the period,
there can be parsing problems or unintended behavior. For example, the following declaration
is syntactically correct but causes an unintended interpretation because of a missing ‘.’, since
this way sorts A, B, sort, and C are declared.
sorts A B
sort C .
Note also that the keywords sort and sorts are synonyms. One may use sort for multiple
sort declarations and sorts for single ones, although we do not encourage this style.
For example, we can declare sorts Zero, NzNat, and Nat in the NUMBERS module, either one
at a time
sort Zero .
sort NzNat .
sort Nat .
or all at once
sorts Zero Nat NzNat .
A sort name can also be structured. Structured sort names are used in parameterized
modules; for example, we may use List{X} for a parameterized list sort with parameter X and
List{Nat} for its instantiation to lists of natural numbers (see Section 7.4.3). A structured sort
name contains at least one pair of curly brace symbols ‘{’ and ‘}’, and is constructed according
to the following BNF grammar, without any white space between terminals:
⟨Sort⟩ ::= ⟨sort identifier⟩
| ⟨Sort⟩ { ⟨SortList⟩ }
⟨SortList⟩ ::= ⟨Sort⟩
| ⟨SortList⟩, ⟨Sort⟩

<!-- manual page 40 -->
28 CHAPTER 3. SYNTAX AND BASIC PARSING
Notice that structured sorts are allowed to contain ‘{’, ‘,’ and ‘}’ but only in accordance
with the above grammar. In fact, several other restrictions apply to sort names and sort
identifiers, namely, the character sequences <, ->, ~>, and @ cannot be used as sort identifiers
and identifiers used for sorts cannot contain any of the characters ‘:’, ‘.’, ‘,’, ‘(’, ‘)’, ‘{’, ‘}’,
‘[’, or ‘]’.2 The reasons for these restrictions will become clear below in this section and in
Sections 3.4, 3.5, and 17.2.1.
Although any so restricted identifier is a legal sort name, the preferred style is to capitalize
the first letter of the name. Furthermore, in the case of a compound name, such as a sort of
nonzero naturals, the names (each with the first letter capitalized) or suitable abbreviations
will be juxtaposed without spaces or hyphens, like, for example, NzNat.
Thus, all the following are structured sort names:
a{X}
a{X, Y}
a{b, c{d}}{e}
while the following are not legal sort names:
{X} (lacks sort identifier prefix)
a(X, Y) a({) (parentheses and ‘,’ not inside braces)
a{b, {d}}{e} ({d} lacks sort identifier prefix)
(parentheses not inside braces and ‘{’ without closing ‘}’)
Structured sort names can be written in an equivalent single-identifier form by using back-
quotes. For example, the sort a{b, c{d}}{e} may be written as a‘{b‘,c‘{d‘}‘}‘{e‘}. Hy-
brid notation such as in a{b‘,c} is not allowed. When backquotes are omitted, the sort name
becomes a sequence of tokens according to Maude’s usual tokenization rules and arbitrary white
space may be inserted between tokens. For example, Foo‘{X‘,Y‘}, Foo{X,Y}, and Foo{X, Y}
are three equivalent forms for the same structured sort name.
Structured sort names must be written in their equivalent single-identifier form inside op-
erator hooks (see Chapter 8) or in metasyntax (see Chapter 17).
Apart from their special syntax and their use as parameterized sorts in parameterized mod-
ules (see Section 7.4.3), structured sort names behave just like sort identifiers.
The subsort relation on sorts parallels the subset relation on the sets of elements in the
intended model of these sorts. Subsort inclusions are declared using the keyword subsort. The
declaration
subsort ⟨Sort-1⟩ < ⟨Sort-2⟩.
states that the first sort is a subsort of the second. For example, the declarations
subsort Zero < Nat .
subsort NzNat < Nat .
specify that the sorts Zero (containing only the constant 0) and NzNat (the nonzero natural
numbers) are subsorts of Nat, the natural numbers. More than one subsort relationship can be
declared using the keyword subsorts, as follows:
subsorts ⟨Sort-1⟩... ⟨Sort-j⟩ < ... < ⟨Sort-k⟩... ⟨Sort-l⟩.
Then, the above declarations can be given in a single declaration as follows:
subsorts Zero NzNat < Nat .
2The characters : and. are allowed in sort identifiers appearing inside curly braces, and ->, ˜>, @, and <
are allowed as sort identifiers appearing inside curly braces.

<!-- manual page 41 -->
3.4. OPERATOR DECLARATIONS 29
If we extend NUMBERS with sorts Int and NzInt we can express the additional subsort
relationships compactly by
sorts NzInt Int .
subsorts NzNat < NzInt Nat < Int .
A set of subsort declarations must define a partial order among the set of sorts. For this to
be true, the user is required to avoid cycles in the subsort declarations. For example, if a sort
A is declared as a subsort of B, and B is declared as a subsort of A, we would have a cycle.
Note that the partial order of subsort inclusions partitions the set of sorts into connected
components, that is, into sets of sorts that are directly or indirectly related in the subsort
ordering. For example, all the above sorts Zero, Nat, NzNat, NzInt, and Int belong to the
same connected component in the subsort ordering, whereas a sort Bool would clearly belong
to a different connected component and could have other sorts, for example a supersort Prop
of propositions, related to it in the same component. Intuitively, connected components gather
together related sorts of data such as numerical data, truth-value data, and so on. Graphically,
we can visualize the partial order of subsort inclusions as an acyclic graph (the corresponding
Hasse diagram), and then the connected components are exactly those of the underlying graph,
as in the following example:
Int
@@
@@
Nat NzInt
Prop
Zero NzNat
Bool
3.4 Operator declarations
In a Maude module, an operator is declared with the keyword op followed by its name, followed
by a colon, followed by the list of sorts for its arguments (called the operator’s arity or domain
sorts), followed by ->, followed by the sort of its result (called the operator’s coarity or range
sort), optionally followed by an attribute declaration (the discussion of operator attributes is
postponed to Section 4.4), followed by white space and a period. Thus the general scheme has
the form
op ⟨OpName⟩ : ⟨Sort-1⟩... ⟨Sort-k⟩ -> ⟨Sort⟩ [⟨OperatorAttributes⟩] .
Here are some operator declarations for our NUMBERS module.
op zero : -> Zero .
op s_ : Nat -> NzNat .
op sd : Nat Nat -> Nat .
ops _+_ _*_ : Nat Nat -> Nat .
If the argument list is empty, the operator is called a constant. Thus zero is a constant.
The name of the operator is a string of characters that may consist of several identifiers,
due to the presence of blanks or other special characters. Underscores (_) play a special role
in these strings. If no underscore character occurs in the operator string—as in the case of the
operator sd above—then the operator is declared in prefix form. If underscore characters occur
in the string, then their number must coincide with the number of sorts declared as arguments
of the operator (in particular, constant names cannot include any underscore character). The
operator is then in mixfix form, with the n-th underscore indicating the place where arguments

<!-- manual page 42 -->
30 CHAPTER 3. SYNTAX AND BASIC PARSING
of the n-th sort must be placed in expressions formed with that operator. In the above example
the operators s_, _+_, and _*_ are in mixfix form.
There may or may not be any other characters before or after any of the underbars. If no
other characters appear, we say that the operator has been declared with empty syntax. For
example, we could declare a sort NatSeq of sequences of natural numbers formed with empty
syntax as follows:
sort NatSeq .
subsort Nat < NatSeq .
op __ : NatSeq NatSeq -> NatSeq [assoc] .
where assoc is an attribute declaring that sequence concatenation is associative (see Sec-
tion 4.4.1). With this operator declaration we can write number sequences such as
zero (s zero) (s s zero)
Operators having the same arity and coarity can be declared simultaneously by using the
keyword ops and giving the non-empty list of their corresponding names after the ops keyword
and before the :, as is done for the declarations of _+_ and _*_ in the example above.
An operator can also be declared using several identifiers. This can be due to the presence
of special characters, or to blank spaces, or both. Consider for example the operator declaration
op [_] and then [_] : Command Command -> Command .
that may allow a natural language style in the syntax of a programming language. It uses
eight identifiers in the Maude sense, but declares a single binary operator, with the underscores
indicating the place of the arguments in the mixfix notation. Internally, Maude also associates
to this operator a corresponding single-identifier form by using backquotes. We could have
equivalently defined the operator using the single-identifier form, namely,
op ‘[_‘]and‘then‘[_‘] : Command Command -> Command .
Of course, both variants are equivalent and have the same mixfix display, but the version
without backquotes is obviously more convenient.3
The declaration of an operator requires an extra pair of parentheses if we already use
parentheses as part of the syntax of the operator. Suppose we had in a programming language
a binary operator (_ only after _). Then, we have to declare it as follows:
op ((_ only after _)) : Command Command -> Command .
Since an operator may be declared using several identifiers, in an ops declaration involving
several operators each operator declaration can be enclosed in parentheses if necessary, to
indicate where the syntax of each operator begins and ends. We could have declared both
operators together, as follows:
ops ([_] and then [_]) ((_ only after _)) :
Command Command -> Command .
Thus, oneorseveralMaudeidentifierscanbeusedinoperatordeclarations. Regardingstyle,
the preferred one, particularly for single-identifier operators with prefix syntax, is to use lower
case names. However, for a composed name such as a meta parse operator, the subsequent
names will be juxtaposed and will typically begin with a capital letter to enhance readability,
e.g., metaParse.
Constants may be given the pconst attribute to mark them as parameter constants (see
Section 7.4.1). Like sorts, they can also be given structured names in parameterized modules
(see Section 7.4.3).
3InFullMaude, operatornamesinoperatordeclarationsmustbegivenassingleidentifiers. Multiple-identifier
names are also supported, but their equivalent single-identifier form must be used in their declarations.

<!-- manual page 43 -->
3.5. KINDS 31
3.5 Kinds
The equational logic underlying Maude is membership equational logic [95, 14]. In this logic
sorts are grouped into equivalence classes called kinds. For this purpose, two sorts are grouped
together in the same equivalence class if and only if they belong to the same connected com-
ponent. Maude sorts are user-defined, while kinds are implicitly associated with connected
components of sorts and are considered as “error supersorts.” Terms (see Section 3.8) that have
a kind but not a sort are understood as undefined or error terms.
In Maude modules, kinds are not independently and explicitly named. Instead, a kind is
identified with its equivalence class of sorts and can be named by enclosing the name of one or
more of these sorts in square brackets [...]; when using more than one sort, they are separated
by commas.
For example, suppose we add a partial predecessor function to our NUMBERS module,
op p : NzNat -> Nat .
Then Maude will parse the term p(zero) and assign it the kind [Nat], or equivalently
[NatSeq] or also [Nat, NatSeq], since the sorts Nat and NatSeq belong to the same connected
component. Although any sort, or list of sorts in the connected component, can be enclosed
in brackets to denote the corresponding kind, Maude uses a canonical representation for kinds;
specifically, Maude prints the kind using a comma-separated list of the maximal elements of
the connected component.
The Maude system also lifts automatically to kinds all the operators involving sorts of the
corresponding connected components to form error expressions. Such error expressions allow
us to give expressions to be evaluated the benefit of the doubt: if, when they are simplified,
they have a legal sort, then they are okay; otherwise, the fully simplified error expression is
returned, which the user can interpret as an error message. Equational simplification can also
occur at the kind level, so that operators can map error terms to defined terms, which may be
useful for error recovery.
It is also possible to explicitly declare operators at the kind level. This corresponds to
declaring a partial operation, which is defined for those argument values for which Maude can
determine that the resulting term has a sort. Note that the operation is considered to be total
at the kind level. As an example, consider the following fragment of a graph specification:
sorts Node Edge .
ops source target : Edge -> Node .
sort Path .
subsort Edge < Path .
op _;_ : [Path] [Path] -> [Path] .
The sorts Node and Edge, along with the source and target operators mapping edges to
nodes, axiomatize the basic graph concepts. The sort Path is intended to be the paths through
the graph, sequences of edges with the target of one edge being the source of the next edge.
Edges are singleton paths, and _;_ denotes the partial concatenation operation, indicated by
giving kinds rather than sorts in the argument list. Later, in Section 4.3, we will see how to
specify when a sequence of edges has sort Path.
To emphasize the fact that an operator defined at the kind level in general defines only a
partial function at the sort level, Maude also supports a notational variant in which an (always
total) operator at the kind level can equivalently be defined as a partial operator between sorts
in the corresponding kinds, with syntax ‘~>’ instead of ‘->’ to indicate partiality. For example,
the above operator declaration can be equivalently specified by
op _;_ : Path Path ~> Path .

<!-- manual page 44 -->
32 CHAPTER 3. SYNTAX AND BASIC PARSING
More generally, the partial operator declaration
op ⟨OpName⟩ : ⟨Sort-1⟩... ⟨Sort-k⟩ ~> ⟨Sort⟩.
is equivalent to the total operator declaration at the kind level
op ⟨OpName⟩ : [⟨Sort-1⟩] ... [⟨Sort-k⟩] -> [⟨Sort⟩] .
3.6 Operator overloading
Operators in Maude can be overloaded, that is, we can have several operator declarations for
the same operator with different arities and coarities. Consider extending our number module
with a new sort Nat3 (of natural numbers modulo 3), constants 0, 1, and 2 of sort Nat3, and
two further operator declarations for _+_.
op _+_ : NzNat Nat -> NzNat .
sort Nat3 .
ops 0 1 2 : -> Nat3 .
op _+_ : Nat3 Nat3 -> Nat3 .
Now _+_ is overloaded, having three declarations. However, there are two different kinds
of overloading present in the example. The additional declaration of _+_ with first argument
NzNat is an example of subsort overloading. Here the two _+_ operators on Nat and NzNat are
supposed to have the same behavior on their shared argument values, that is, the operator on
the subsort NzNat is the restriction of the operator on the larger sort Nat. The main point
of such declarations is to give more sort information, for example that the result of adding a
nonzero natural number to any natural number is nonzero. Many more examples of this form of
overloading can be found in the predefined data modules for the number hierarchy (Chapter 8)
and in other modules throughout the manual.
In contrast, the sorts Nat and NzNat on the one hand, and the sort Nat3 on the other
belong to two different connected components in the subsort ordering and therefore natural
number addition and addition modulo 3 are semantically unrelated. This form of overloading
is called ad-hoc overloading. Both subsort and ad-hoc overloading of operators are allowed in
Maude. However, to avoid ambiguous expressions we require that if the sorts in the arities of
two operators with the same syntactic form are pairwise in the same connected components,
then the sorts in the coarities must likewise be in the same connected component.
Strictly speaking, this requirement would rule out ad-hoc overloaded constants. For this
reason, wehavedeclaredtwodifferentconstantszeroand0forthecorrespondingzeroelements.
However, this requirement can be relaxed, and it is often natural to do so. For example, the
constants of a parameterized module (see Chapter 7.4) can appear in many different connected
components for different instances of the module, and it may be cumbersome to rename them
all. To allow this relaxation, constants—and, more generally, terms (see Section 3.8)—can be
qualified by their sort, by enclosing them in parentheses followed by a dot and the sort name.
In this way, we could have instead declared 0 as an ad-hoc overloaded constant for natural
numbers and for natural numbers modulo 3, and could then disambiguate the expression 0 + 0
by writing, for example, 0 + (0).Nat and 0 + (0).Nat3, or (0 + 0).Nat and (0 + 0).Nat3.
3.7 Variables
A variable is constrained to range over a particular sort or kind. Variables can be declared
on-the-fly in Maude with syntax consisting of an identifier (the variable name), a colon, and

<!-- manual page 45 -->
3.8. TERMS AND PREREGULARITY 33
another identifier (its sort) or kind expression (its kind). For example, N:Natdeclares a variable
named N of sort Nat, and X:[Nat] declares a variable named X of kind [Nat].
The scope of an on-the-fly variable declaration is the declaration’s occurrence. Thus each
such variable must be accompanied by its sort or kind.
A variable can also be declared in a module using the keyword var followed by an identifier
(the variable name), followed by a colon with white space before and after, followed by an
identifier (its sort) or kind expression (its kind), followed by white space and a period.
var N : Nat .
var X : [Nat] .
Thescopeofsuchadeclarationistheentiremodule. Ithastheeffectofreplacingoccurrences
of N and X by the on-the-fly versions N:Nat and X:[Nat].
Multiple variables of the same sort can be declared using the keyword vars.
vars M N : Nat .
vars X Y : [Nat] .
Both upper and lower case names for variables are possible. However, upper case variable
names are more customary in Maude. The syntactic conventions for the acceptable names
of variables in variable declarations are the same as those for constant operators, that is, for
operators with empty arity. In particular, the underscore ‘_
’ cannot be used in the name of a
variable, but the colon ‘:’ can; thus the scanning for ‘:’ in order to extract the appropriate
sort or kind from an on-the-fly variable declaration is done from right to left.
3.8 Terms and preregularity
A term is either a constant, a variable, or the application of an operator to a list of argument
terms. The sort of a constant or variable is its declared sort. In the application of an operator,
the argument list must agree with the declared arity of the operator. That is, it must be of the
same length, and each term must have sort (or at least kind) in the connected component of the
corresponding declared argument sort. Using prefix form—which can always be used for any
operator, regardless of having been declared with either prefix or mixfix syntax—the syntax
of operator application is the operator’s name followed by ‘(’, followed by a list of argument
terms separated by commas, followed by ‘)’. Here are some examples of prefix notation from
our numbers module.
s_(zero)
s_(sd(N:Nat, M:Nat))
p(s_(zero))
_+_(N:Nat, M:Nat)
The application of an operator declared with mixfix form also has a mixfix syntax: the
operator’s mixfix name with each underscore replaced by the corresponding term from the
argument list. The mixfix form of the above examples is
s zero
s sd(N:Nat, M:Nat)
p(s zero)
N:Nat + M:Nat
The kind of a term is the result kind of its topmost operator. For example, the kind of
p(s zero) is [Nat], since Nat is the result sort of p. If a module’s grammar is unambiguous
(see the discussion on parsing in the following section), then each term has a single kind. But

<!-- manual page 46 -->
34 CHAPTER 3. SYNTAX AND BASIC PARSING
we can also associate sorts to terms. In general, even if the grammar is unambiguous, a term
may have several sorts, due to the subsort ordering. Specifically, constants have the sort they
are declared with and any supersort of it. Given a term of the form f(t1,...,tn), if ti has
sort si for i = 1,...,n and there is an operator declaration f : s1 ...sn →s, then the term
f(t1,...,tn) has sort sand any of its supersorts. For example, in our example NUMBERS module
the term s s 0 has sorts NzNat and Nat.
A very desirable property of a module is that each term has a least sort that can be assigned
to it. Such a least sort gives us the most detailed information on how to classify such a term as a
data element. For example, the least sort of the term s s 0 is NzNat, and this gives us the most
precise classification of such a term in the sort hierarchy. Given an arbitrary signature Σ, we
can have terms that fail to have a least sort. However, if Σ satisfies a simple syntactic property
called preregularity [68], we can guarantee that any Σ-term will have a least sort. We call Σ
preregular if for each n, given an n-argument function symbol f and sorts s1,...,sn such that
f(x1 : s1,...,xn : sn) is a well-formed Σ-term, then there is a least sort s among all the sorts
s′appearing in (possibly overloaded) operator declarations of the form f : s′
1,...,s′
n −→s′in
Σ such that for 1 ≤i≤n we have si ≤s′
i. For example, the signature
sorts A B C D .
subsorts A < B C < D .
op a : -> A .
op f : B -> B .
op f : C -> C .
fails to be preregular, because for the sort A the term f(X:A) is a well-formed term, but there
is no least sort for the result of f with arguments greater or equal to A, since either B or C can
be chosen as result sorts, and they are incomparable in the sort hierarchy. As a consequence,
both f(X:A) and f(a) do not have a least sort: they have sorts B, C, and D, and B and C are
minimal sorts among those sorts.
As already mentioned in Section 3.4 for the assocattribute and further explained in Section
4.4.1, operators can be declared with equational axioms such as associativity (assoc), commu-
tativity (comm), and identity (id:). This means that, if we denote by A the corresponding
associativity and/or commutativity, and/or identity equations, we are not really interested in
syntactic terms t, but rather in equivalence classes modulo A, that is, in the equivalence class
[t]A of each term t, since all representatives of the class are viewed as equivalent representations.
Preregularity modulo A now means that we can assign a least sort not just to any well-formed
term t, but also to its equivalence class [t]A. As further explained in Section 21.3.5, Maude
assumes that modules are preregular modulo whatever axioms such as assoc, comm, and id:
have been declared for operators, checks syntactic conditions ensuring preregularity modulo A,
and generates warnings when a module fails to satisfy such preregularity conditions.
A ground term is a term containing no variables: only constants and operators. Intuitively,
ground terms denote either data in case no equations apply to the term (for example, s zero
is data) or functional expressions indicating how an equationally defined function is applied
to data (for example, (s zero) + (s zero)). Ground terms modulo equations constitute the
initial algebra associated with a specification, as discussed later in Section 4.3.
3.9 Parsing
As seen in previous sections, the Maude language supports user-definable syntax including
mixfix operator declarations. Parsing is done in stages. First Maude’s surface syntax is parsed
using a bison/flex-based parser. Operator declarations are extracted and are used to construct

<!-- manual page 47 -->
3.9. PARSING 35
a grammar for language constructs that can contain terms. Finally those constructs are parsed
using a version of Leo’s algorithm for context-free grammars [80], which has been extended to
handle Maude-specific features including the precedence-gather mechanism and bubbles.
With mixfix syntax, the occurrence of ambiguities in the parsing of terms is very common.
Of course, we can always provide unambiguous grammars, which are frequently surprisingly
large, or use parentheses for breaking the possible ambiguities. But usually we would like to
have a more powerful alternative. Maude reduces such ambiguities by using a mechanism based
on precedence values and gathering patterns.
Let us assume the following declarations for some arithmetic expressions:
sort Nat .
ops 1 2 3 : -> Nat .
ops _+_ _*_ : Nat Nat -> Nat .
An expression like 1 + 2 * 3 is ambiguous, since both (1 + 2) * 3 and 1 + (2 * 3) are
valid parses. This kind of ambiguity is usually solved by assigning a precedence to each of the
operators. In Maude, the precedence of an operator is given by a natural number,4 where a
lower value indicates a tighter binding.
Operatorprecedencethendefineshowanexpressionshouldbeparsedwhenseveraloperators
are present. We can assign a precedence to an operator with a precedence (abbreviated prec)
attribute, which takes the precedence value as an argument. For example, one would expect
multiplication to be evaluated before addition. Thus, we can give precedences, e.g., 33 and 31
to the operators _+_ and _*_, respectively, as follows:
op _+_ : Nat Nat -> Nat [prec 33] .
op _*_ : Nat Nat -> Nat [prec 31] .
The term 1 + 2 * 3 is now unambiguous: its only possible parse is 1 + (2 * 3).
Precedence can be overridden using parentheses; we can always write (1 + 2) * 3 in case
this is the term we are interested in. For those operators for which the user does not specify
a precedence value, a default one is given (see Section 3.9.1 for a discussion on the default
precedence values). For example, both operators _+_ and _*_ above get 41 as their default
precedence, and hence the ambiguity.
The precedence mechanism is not enough, however. For example, the expression 1 + 2 + 3
is still ambiguous, because both parses (1 + 2) + 3 and 1 + (2 + 3) are possible. Usually,
programming languages define a way of associating operators to solve this kind of problems,
so that the associativity of the operators determines which is evaluated first. For example,
addition usually is left-associative, and therefore we expect to parse it as (1 + 2) + 3. In
Maude, we can specify not only the associativity of operators, but general gathering patterns
for each operator.
The gathering pattern of an operator restricts the precedences of terms that are allowed as
arguments. We give a (non-empty) sequence of as many E, e, or & values as the number of
arguments in the operator, that is, one of these values for each argument position:
• E indicates that the argument must have a precedence value lower or equal than the
precedence value of the operator,
• e indicates that the argument must have a precedence value strictly lower than the prece-
dence value of the operator, and
• &indicates that the operator allows any precedence value for the corresponding argument.
Infact, theprecedencevaluesworkbecauseoftheircombinationwiththegatheringpatterns.
For example, the precedence values given to _+_ and _*_ work as expected because their default
gathering pattern is (E E) (see Section 3.9.2), which forces them to be applied only to terms
4The maximum allowed precedence value is 231
−1.

<!-- manual page 48 -->
36 CHAPTER 3. SYNTAX AND BASIC PARSING
of smaller or equal precedence value. Thus, 1 + (2 * 3) is a valid parse for 1 + 2 * 3. On
the other hand, since the precedence of a term is given by the precedence of its top operator,
(1 + 2) * 3 is not a valid parse for 1 + 2 * 3, because the term 1 + 2 has precedence value
33, which is greater than the precedence of _*_.
Moreover, by default, all constants have precedence 0 (see Section 3.9.1), and therefore they
are also valid arguments for both operators.
We can specify _+_ and _*_ as left-associative by giving to them gathering pattern (E e).
op _+_ : Nat Nat -> Nat [prec 33 gather (E e)] .
op _*_ : Nat Nat -> Nat [prec 31 gather (E e)] .
In this way, we force the second argument of these operators to be of a strictly lower precedence.
Then, a term with _+_ as top operator (or any other operator with the same precedence)
like 2 + 3 is nonvalid as second argument for _+_. But it would be valid as first argument,
since terms with equal precedence are allowed. Now the only possible parse for the expression
1 + 2 + 3 is (1 + 2) + 3.
Notethatparenthesescouldbedescribedasanoperator(_)withprecedence0 andgathering
pattern (&). Thus, any term can appear inside parentheses, and any subterm of a term can be
enclosed in parentheses.
3.9.1 Default precedence values
Maude associates default precedence values to those operators for which the user does not
specify this information as part of the operator declaration. The default precedence values are
entirely similar to those used by OBJ3 [69]. The rules for the assignment of default precedence
values are:
• Operators with standard form (constants and prefix operators) always have precedence
0, regardless of user settings. The user cannot change the precedence value or gathering
pattern for operators in standard form.
• Mixfix operators which begin and end with something different from an underbar have
precedence 0. Operators as, for example, (_), <_:_|_>, and if_then_else_fi follow this
rule.
• Mixfix operators which begin or end with an underbar have precedence 15 for a unary
operator and 41 for everything else. Note that this ‘or’ is exclusive. Operators like, e.g.,
not_, _!, or to_:_ fall into this category.
• Mixfix operators which begin and end with an underbar have precedence 41. This rule
applies, e.g., to the operators__, _+_, _*_, and _?_:_.
3.9.2 Default gathering patterns
As for precedence values, Maude assigns default gathering patterns to all those operators for
which the user does not specify this information as part of the operator declaration. The
default gathering patterns are also entirely similar to those used by OBJ3 [69]. The rules for
the assignment of the default gathering patterns are:
• All arguments of prefix operators have a gathering value &, regardless of the user specifi-
cation.
• If the underbar corresponding to an argument is not adjacent to another underbar, and it
is neither the leftmost nor the rightmost token in the operator, then the default gathering
valueforsuchanargumentis&. Inotherwords, ifanunderbarappearsbetweentokensdif-
ferent from the underbar, then its corresponding argument will have this default gathering
pattern. For example, the default gathering pattern for the operator if_then_else_fi

<!-- manual page 49 -->
3.9. PARSING 37
is (& & &), the default gathering pattern for the operator [_and then_] is (& &), and
the default gathering pattern for the operator (_) is (&).
• If the underbar corresponding to an argument is adjacent to another underbar, or if it
is the leftmost or the rightmost token in the operator, then the default gathering value
for such an argument is E. Thus, e.g., the default gathering pattern for the operator
not_ is (E), the default gathering pattern for the operator _?_:_ is (E & E), the default
gathering pattern for the operator _+_ is (E E), and the default gathering pattern for the
operator__
is (E E).
Those binary operators which start with an underscore, end with an underscore, and have
a precedence greater than 0 are handled as special cases:
– The operator will have gathering pattern (e E) if it has the assoc attribute (see
Section 4.4.1). For example, the following operators fall into this category.
op _+_ : Nat Nat -> Nat [assoc] .
op _*_ : Nat Nat -> Nat [assoc] .
op __ : NatList NatList -> NatList [assoc] .
– If the operator does not have the assoc attribute, but its first argument, its last
argument, and its coarity are in the same connected component of sorts, then:
1. if the subsort relations allow it to right-associate but not left-associate, then the
first argument’s gathering pattern will change to e, and
2. if the subsort relations allow it to left-associate but not right-associate, then the
last argument’s gathering pattern will change to e.
Assuming Int < IntList, then the operators
op _<:_ : Int IntList -> IntList .
op _:>_ : IntList Int -> IntList .
have, by default, gathering patterns (e E)and (E e), respectively. According to the
general rule, since their argument bars are the leftmost and the rightmost tokens, the
gathering pattern should be (E E) for both of them. However, both operators fall
into the second special case, since they are binary operators which start and end with
underscores, have a precedence greater than 0 (by default 41), and are not declared
associative. Given the subsort relation, the operator _<:_ may right-associate, but
not left-associate, that is, 1 <: 2 <: 3 should be parsed as 1 <: (2 <: 3), but
(1 <: 2) <: 3 should not be a valid parse. Therefore, _<:_ gets default gathering
pattern (e E). And similarly for _:>_, although in this case it can left-associate,
and therefore it gets default gathering pattern (E e).
3.9.3 The extended signature of a module
In addition to the signature defined by the user, parsing of terms takes place in an ex-
tended grammar in which information for handling parentheses, sort and equality predicates,
if_then_else_fi, and qualification operators are included. These structures belong to the
so-called extended signature of a module. The main structures added in the extended signature
of a module are:
• Sort disambiguation. For each sort S in the signature of a module, Maude adds to the
signature the operator
op (_).S : S -> S .
This helps in the disambiguation of ad-hoc overloaded constants and terms. As an exam-
ple, remember from Section 3.6 that if we declare 0 as an ad-hoc overloaded constant for

<!-- manual page 50 -->
38 CHAPTER 3. SYNTAX AND BASIC PARSING
natural numbers and for natural numbers modulo 3, then we can disambiguate the expres-
sion 0 + 0 by writing, for example, 0 + (0).Nat and 0 + (0).Nat3, or (0 + 0).Nat
and (0 + 0).Nat3. As another example, in the module META-MODULE (see Section 17.4),
the term none is ambiguous, since the operator none is used as the empty set of operator
declarations, equations, rules, etc. We can disambiguate it by writing (none).OpDeclSet.
Of course, these disambiguation operators can be used not only for constants, but for any
term. For example, we can write (2 + 3).Nat as a valid term in the predefined module
NAT.
• Parentheses. The extended signature of a module contains the operator
op (_) : S -> S .
for each sort S in its signature. These operators allow the use of parentheses without hav-
ing to declare a parentheses operator for each sort. For example, (2 + 3), (2 + 3) + 5,
(2 + (3) + 5), (((2 + 3)) + 5), are all valid terms in NAT, thanks to these declara-
tions.
• Equivalent single-identifier form foralloperators. Eachdeclaredoperator, includingthose
in mixfix form, may also be used in their equivalent single-identifier prefix form. For
example, in the NAT module, the term _+_(2, 3) is equivalent to 2 + 3, and the terms
if true then 2 + 3 else - 3 fi and if_then_else_fi(true, _+_(2, 3), -_(3))
are equivalent; any combination is possible so if_then_else_fi(true, 2 + 3, - 3) is
also valid.
• Flattened associative argument lists. Operators with the attribute assoc may be used in
Maude in a nonparenthesized flattened form (see Section 4.8). This is possible thanks to
the precedence-gathering values in mixfix notation, but it is also possible in prefix syntax.
For example, gcd(2, 3, 4) is a valid term in NAT, where gcd is the greater common divi-
sor operator, which is declared as a binary associative operator. Of course, this term can
always be written in the standard format as gcd(2, gcd(3, 4)) or gcd(gcd(2, 3), 4).
Furthermore, wecancombinethispossibilitywiththesingle-identifierformtowritethings
like _+_(2, 3, 4) instead of _+_(_+_(2, 3), 4) or _+_(2, _+_(3, 4)), but of course,
since _+_ is declared with the assoc attribute in the predefined module NAT, we can just
write 2 + 3 + 4.
• Polymorphic operators and the BOOL module. All the information contained in the prede-
finedmodulesTRUTH-VALUE,TRUTH,BOOL-OPS,andBOOLisincludedintheextendedsigna-
tureofeachmodule(unlessthisinclusionisexplicitlydisabled). Inparticular, appropriate
instances of the polymorphic operators contained in TRUTH (that is, if_then_else_fi,
_==_, and _=/=_) are generated for each sort in the module; in addition, for each sort S,
a sort predicate _:: S is also added. All these modules and operators are fully explained
in Section 8.1.
3.9.4 Parsing examples
Maude provides the parse command for parsing terms. The command does not do anything
other than parsing the given term in the extended signature of the module. This is exactly what
is done when a term appears in a command, before executing such a command. For example,
when we try to reduce a term (2 + 3) * 5, the system first parses it and then reduces it. If
the term is ambiguous, or there is no parse for it, an error message is given and no further
action takes place.
Maude> reduce in NAT : 2 + true .
Warning: <standard input>, line 1:
didn’t expect token true: 2 + true <---*HERE*

<!-- manual page 51 -->
3.9. PARSING 39
Warning: <standard input>, line 1: no parse for term.
For testing the parsing of terms we can use the parse command.
Maude> parse in NAT : 2 + true .
Warning: <standard input>, line 1:
didn’t expect token true: 2 + true <---*HERE*
Warning: <standard input>, line 1: no parse for term.
As other commands, parsing can take place either in the module explicitly mentioned in the
command or in the current module.
We illustrate the use of the parse command for the examples introduced in the previous
sections. Let us first consider a module PARSING-EX1 with constants 1, 2, and 3, and binary
operators _+_ and _*_.
fmod PARSING-EX1 is
sort Nat .
ops 1 2 3 : -> Nat .
ops _+_ _*_ : Nat Nat -> Nat .
endfm
Since_+_and_*_aredeclaredwithoutprecedencevalues, andthereforebothgetthedefault
value 41, we obtain the following result.
Maude> parse 1 + 2 * 3 .
Warning: <standard input>, line 13: ambiguous term, two parses are:
1 + (2 * 3) -versus- (1 + 2) * 3
Arbitrarily taking the first as correct. Nat: 1 + (2 * 3)
As a first solution, we may consider using parentheses.
Maude> parse in PARSING-EX1 : 1 + (2 * 3) .
Nat: 1 + (2 * 3)
Maude> parse in PARSING-EX1 : (1 + 2) * 3 .
Nat: (1 + 2) * 3
Let us now consider the module PARSING-EX2, where _+_ and _*_ are declared with prece-
dences 33 and 31, respectively.
fmod PARSING-EX2 is
sort Nat .
ops 1 2 3 : -> Nat .
op _+_ : Nat Nat -> Nat [prec 33] .
op _*_ : Nat Nat -> Nat [prec 31] .
endfm
Now, parentheses are not necessary for parsing the term 1 + 2 * 3.
Maude> parse in PARSING-EX2 : 1 + 2 * 3 .
Nat: 1 + 2 * 3
Of course, we may still use parentheses.
Maude> parse in PARSING-EX2 : (1 + 2) * 3 .
Nat: (1 + 2) * 3
Since the default gathering patterns for binary operators like _+_ and _*_ is (E E), a term
like 1 + 2 + 3 is ambiguous.

<!-- manual page 52 -->
40 CHAPTER 3. SYNTAX AND BASIC PARSING
Maude> parse in PARSING-EX2 : 1 + 2 + 3 .
Warning: <standard input>, line 30: ambiguous term, two parses are:
1 + (2 + 3) -versus- (1 + 2) + 3
Arbitrarily taking the first as correct. Nat: 1 + (2 + 3)
As above, we may use parentheses to parse such terms.
Maude> parse in PARSING-EX2 : (1 + 2) + 3 .
Nat: (1 + 2) + 3
Maude> parse in PARSING-EX2 : 1 + (2 + 3) .
Nat: 1 + (2 + 3)
Let us now consider the module PARSING-EX3, where _+_ and _*_ are declared to be left-
associative, that is, with gathering patterns (E e).
fmod PARSING-EX3 is
sort Nat .
ops 1 2 3 : -> Nat .
op _+_ : Nat Nat -> Nat [prec 33 gather (E e)] .
op _*_ : Nat Nat -> Nat [prec 31 gather (E e)] .
endfm
Now, the terms above have unambiguous parses.
Maude> parse in PARSING-EX3 : 1 + 2 * 3 .
Nat: 1 + 2 * 3
Maude> parse in PARSING-EX3 : 1 + 2 + 3 .
Nat: 1 + 2 + 3
Let us now consider the module PARSING-EX4, where _+_ and _*_ are declared to be asso-
ciative. Note that in this case, by default, they are assigned gathering patterns (E e).
fmod PARSING-EX4 is
sort Nat .
ops 1 2 3 : -> Nat .
op _+_ : Nat Nat -> Nat [prec 33 assoc] .
op _*_ : Nat Nat -> Nat [prec 31 assoc] .
endfm
Maude> parse in PARSING-EX4 : 1 + 2 * 3 .
Nat: 1 + 2 * 3
Maude> parse in PARSING-EX4 : 1 + 2 + 3 .
Nat: 1 + 2 + 3
We illustrate the use of the extended signature in which all terms are parsed with the
following examples.
Maude> parse in PARSING-EX1 : (2 + 3).Nat .
Nat: 2 + 3
Maude> parse in PARSING-EX1 : (2).Nat + 3 .
Nat: 2 + 3

<!-- manual page 53 -->
3.9. PARSING 41
Maude> parse in PARSING-EX1 : (2).Nat + (3).Nat .
Nat: 2 + 3
Maude> parse in PARSING-EX1 : ((1) + ((2) + (3))) .
Nat: 1 + (2 + 3)
Maude> parse in PARSING-EX1 : _+_(1, _+_(2, 3)) .
Nat: 1 + (2 + 3)
Maude> parse in PARSING-EX4 : _+_(1, 2, 3) .
Nat: 1 + 2 + 3
Maude> parse in PARSING-EX4 : if 1 == 2 then 1 + 2 else _+_(1, 2) fi .
Nat: if 1 == 2 then 1 + 2 else 1 + 2 fi
Maude> parse in PARSING-EX4 :
if _==_(1, 2)
then if_then_else_fi(1 + 2 :: Nat, 1 * 1, 2 * 1)
else _+_(1, 2)
fi .
Nat: if 1 == 2
then if (1 + 2) :: Nat
then 1 * 1
else 2 * 1
fi
else 1 + 2
fi

<!-- manual page 54 -->
42 CHAPTER 3. SYNTAX AND BASIC PARSING

