# Chapter 1: Introduction (manual pages 13-30)

<!-- manual page 13 -->
Chapter 1
Introduction
This introduction tries to give the big picture on the goals, design philosophy, logical founda-
tions, applications, and overall structure of Maude. It is written in an impressionistic, conver-
sational style, and should be read in that spirit. The fact that occasionally some particular
technical concept mentioned in passing (for example, “the Church-Rosser property”) may be
unfamiliar should not be seen as an obstacle. It should be taken in a relaxed, sporting spirit:
those things will become clearer in the body of the manual; here it is just a matter of gaining
a first overall impression.
1.1 Simplicity, expressiveness, and performance
Maude’s language design can be understood as an effort to simultaneously maximize three
dimensions:
• Simplicity: programs should be as simple as possible and have clear meaning.
• Expressiveness: a very wide range of applications should be naturally expressible: from
sequential, deterministic systems to highly concurrent nondeterministic ones; from small
applications to large systems; and from concrete implementations to abstract specifi-
cations, all the way to logical frameworks, in which not just applications, but entire
formalisms, other languages, and other logics can be naturally expressed.
• Performance: concrete implementations should yield system performance competitive
with other efficient programming languages.
Although simplicity and performance are natural allies, maximizing expressiveness is per-
haps the key point in Maude’s language design. Languages are after all representational devices,
and their merits should be judged on the degree to which problems and applications can be
represented and reasoned about generally, naturally, and easily. Of course, domain-specific lan-
guages also have an important role to play in certain application areas, and can offer a useful
“economy of representation” for a given area. In this regard, Maude should be viewed as a
high-performance metalanguage, through which many different domain-specific languages can
be developed.
1.1.1 Simplicity
Maude’s basic programming statements are very simple and easy to understand. They are
equations and rules, and have in both cases a simple rewriting semantics in which instances of
the lefthand side pattern are replaced by corresponding instances of the righthand side.
1

<!-- manual page 14 -->
2 CHAPTER 1. INTRODUCTION
A Maude program containing only equations is called a functional module. It is a functional
program defining one or more functions by means of equations, used as simplification rules. For
example, if we build lists of quoted identifiers (which are sequences of characters starting with
the character ‘’’ and belong to a sort1 Qid) with a “cons” operator denoted by an infix period,
op nil : -> List .
op _._ : Qid List -> List .
then we can define a length function and a membership predicate by means of the operators
and equations
op length : List -> Nat .
op _in_ : Qid List -> Bool .
vars I J : Qid .
var L : List .
eq length(nil) = 0 .
eq length(I . L) = s length(L) .
eq I in nil = false .
eq I in J . L = (I == J) or (I in L) .
where s_ denotes the successor function on natural numbers, _==_ is the equality predicate
on quoted identifiers, and _or_ is the usual disjunction on Boolean values. Such equations
(specified in Maude with the keyword eq and ended with a period) are used from left to right
as equational simplification rules. For example, if we want to evaluate the expression
length(’a . ’b . ’c . nil)
we can apply the second equation for length to simplify the expression three times, and then
apply the first equation once to get the final value s s s 0:
length(’a . ’b . ’c . nil)
= s length(’b . ’c . nil)
= s s length(’c . nil)
= s s s length(nil)
= s s s 0
Thisisthestandard“replacementofequalsbyequals” useofequationsinelementaryalgebra
and has a very clear and simple semantics in equational logic. Replacement of equals by
equals is here performed only from left to right and is then called equational simplification
or, alternatively, equational rewriting. Of course, the equations in our program should have
good properties as “simplification rules” in the sense that their final result exists and should be
unique. This is indeed the case for the two functional definitions given above.
In Maude, equations can be conditional; that is, they may only be applied if a certain
condition holds. For example, we can simplify a fraction to its irreducible form using the
conditional equation
vars I J : NzInt .
ceq J / I = quot(J, gcd(J, I)) / quot(I, gcd(J, I))
if gcd(J, I) > s 0 .
1In Maude, types come in two flavors, called sorts and kinds (see Section 3, and the discussion of user-
definable data in Section 1.1.2 below).

<!-- manual page 15 -->
1.1. SIMPLICITY, EXPRESSIVENESS, AND PERFORMANCE 3
where ceqis the Maude keyword introducing conditional equations, NzIntis the sort of nonzero
integers, and where we assume that the integer quotient (quot) and greatest common divisor
(gcd) operations have already been defined by their corresponding equations.
A Maude program containing rules and possibly equations is called a system module. Rules
are also computed by rewriting from left to right, that is, as rewrite rules, but they are not
equations; instead, theyareunderstoodaslocal transition rules inapossiblyconcurrentsystem.
Consider, for example, a distributed banking system in which we envision the account objects
as floating in a “soup,” that is, in a multiset or bag of objects and messages. Such objects and
messages can “dance together” in the distributed soup and can interact locally with each other
according to specific rewrite rules. We can represent a bank account as a record-like structure
with the name of the object, its class name (Account) and a bal(ance) attribute, say, a natural
number. The following are two different account objects in our notation:
< ’A-001 : Account | bal : 200 >
< ’A-002 : Account | bal : 150 >
Accounts can be updated by receiving different messages and changing their state accord-
ingly. For example, we can have debit and credit messages, such as
credit(’A-002, 50)
debit(’A-001, 25)
We can think of the “soup” as formed just by “juxtaposition” (with empty syntax) of objects
and messages. For example, the above two objects and two messages form the soup
< ’A-001 : Account | bal : 200 >
< ’A-002 : Account | bal : 150 >
credit(’A-002, 50)
debit(’A-001, 25)
in which the order of objects and messages is immaterial. The local interaction rules for
crediting and debiting accounts are then expressed in Maude by the rewrite rules
var I : Qid .
vars N M : Nat .
rl < I : Account | bal : M > credit(I, N)
=> < I : Account | bal : (M + N) > .
crl < I : Account | bal : M > debit(I, N)
=> < I : Account | bal : (M - N) >
if M >= N .
where rules are introduced with the keyword rl and conditional rules (like the above rule for
debit that requires the account to have enough funds) with the crl keyword.
Notethattheserulesare notequations atall: theyarelocaltransitionrulesofthedistributed
banking system. They can be applied concurrently to different fragments of the soup. For
example, applying both rules to the soup above we get the new distributed state:
< ’A-001 : Account | bal : 175 >
< ’A-002 : Account | bal : 200 >
Note that the rewriting performed is multiset rewriting, so that, regardless of where the
account objects and the messages are placed in the soup, they can always come together and
rewrite if a rule applies. In Maude this is specified in the equational part of the program (system
module) by declaring that the (empty syntax) multiset union operator satisfies the associativity
and commutativity equations:

<!-- manual page 16 -->
4 CHAPTER 1. INTRODUCTION
X (Y Z) = (X Y) Z
X Y = Y X
This is not done by giving the above equations explicitly. It is instead done by declaring
the multiset union operator with the assoc and comm equational attributes (see Section 4.4.1
and Section 1.1.2 below), as follows, where Configuration denotes the multisets or soups of
objects and messages.
op __ : Configuration Configuration -> Configuration [assoc comm] .
Maude then uses this information to generate a multiset matching algorithm, in which the
multiset union operator is matched modulo associativity and commutativity.
Again,aprograminvolvingsuchrewriterulesisintuitivelyverysimple,andhasaverysimple
rewriting semantics. Of course, the systems specified by such rules can be highly concurrent
and nondeterministic; that is, unlike for equations, there is no assumption that all rewrite
sequences will lead to the same outcome. For example, depending on the order in which debit
or credit messages are consumed, a bank account can end up in quite different states, because
the rule for debiting can only be applied if the account balance is big enough. Furthermore,
for some systems there may not be any final states: their whole point may be to continuously
engage in interactions with their environment as reactive systems.
1.1.2 Expressiveness
The above examples illustrate a general fact, namely, that Maude can express with equal ease
andnaturalnessdeterministic computations, whichleadtoauniquefinalresult, andconcurrent,
nondeterministic computations. The first kind is typically programmed with equations in
functional modules, and the second with rules (and perhaps with some equations for the “data”
part) in system modules.
In fact, functional modules define a functional sublanguage2 of Maude. In a functional
language true to its name, functions have unique values as their results, and it is neither easy
nor natural to deal with highly concurrent and nondeterministic systems while keeping the
language’sfunctionalsemantics. Itiswellknownthatsuchsystemsposeaseriousexpressiveness
challenge for functional languages. In Maude this challenge is met by system modules, which
extend the purely functional semantics of equations to the concurrent rewriting semantics of
rules.3 Although certainly declarative in the sense of having a clear logical semantics, system
modules are of course not functional: that is their entire raison d’être.
Besides this generality in expressing both deterministic and nondeterministic computations,
further expressiveness is gained by the following features:
• equational pattern matching,
• user-definable syntax and data,
• types, subtypes, and partiality,
• generic types and modules,
• support for objects, and
• reflection.
We briefly discuss each of these features in what follows.
2This sublanguage is essentially an extension of the OBJ3 equational language [69], which has greatly influ-
enced the design of Maude.
3As explained in Section 1.2, mathematically this is achieved by a logic inclusion, in which the functional
world of equational theories is conservatively embedded in the nonfunctional, concurrent world of rewrite theo-
ries.

<!-- manual page 17 -->
1.1. SIMPLICITY, EXPRESSIVENESS, AND PERFORMANCE 5
Equational pattern matching
Rewriting with both equations and rules takes place by matching a lefthand side term against
the subject term to be rewritten. The most common form of matching is syntactic matching, in
whichthelefthandsidetermismatchedasatreeonthe(treerepresentationofthe)subjectterm
(see Section 4.7). For example, the matching of the lefthand sides for the equations defining
the length and _in_ functions above is performed by syntactic matching. But we have already
encountered another, more expressive, form of matching, namely, equational matching in the
bank accounts example: the lefthand side
< I : Account | bal : M > credit(I, N)
has the (empty syntax) multiset union operator__ as its top operator, but, thanks to its assoc
and comm equational attributes, it is matched not as a tree, but as a multiset. Therefore,
the match will succeed provided that the subject multiset contains instances of the terms
< I : Account | bal : M > and credit(I, N) in which the variable I is instantiated the
same way in both terms, regardless of where those instances appear in the multiset, that is,
modulo associativity and commutativity.
Ingeneral, abinaryoperatordeclaredinaMaudemodulecanbedefinedwithany4 combina-
tion of equational attributes of: associativity, commutativity, left-, right-, or two-sided identity,
and idempotency. Maude then generates an equational matching algorithm for the equational
attributes of the different operators in the module, so that each operator is matched modulo its
equational attributes. This manual will illustrate with various examples the expressive power
afforded by this form of equational matching (see Section 4.8).
User-definable syntax and data
In Maude the user can specify each operator with its own syntax, which can be prefix, postfix,
infix, or any “mixfix” combination. This is done by indicating with underscores the places where
the arguments appear in the mixfix syntax. For example, the infix list cons operator above is
specifiedby_._, the(emptysyntax)multisetunionoperatorby__, andtheif-then-elseoperator
by if_then_else_fi. In practice, this improves readability (and therefore understandability)
of programs and data. In particular, for metalanguage uses, in which another language or logic
is represented in Maude, this can make a big difference for understanding large examples, since
the Maude representation can keep essentially the original syntax. The combination of user-
definablesyntaxwithequationsandequationalattributesformatchingleadstoaveryexpressive
capability for specifying any user-definable data. It is well known that any computable data
type can be equationally specified [9]. Maude gives users full support for this equational style
of defining data which is not restricted to syntactic terms (trees) but can also include lists
(modulo associativity), multisets (modulo associativity and commutativity), sets (adding an
idempotency equation), and other combinations of equational attributes that can then be used
in matching. This great expressiveness for defining data is further enhanced by Maude’s rich
type structure, as explained below.
Types, subtypes, and partiality
Maudehastwovarietiesoftypes: sorts, whichcorrespondtowell-defineddata, andkinds, which
may contain error elements. Sorts can be structured in subsort hierarchies, with the subsort
relation understood semantically as subset inclusion. For example, for numbers we can have
subsort inclusions
Nat < Int < Rat
4Except for any combination including both associativity and idempotency, which is not currently supported.

<!-- manual page 18 -->
6 CHAPTER 1. INTRODUCTION
indicating that the natural numbers are contained in the integers, and these in turn are con-
tained in the rational numbers. All these sorts determine a kind (say the “number kind”) which
is interpreted semantically as the set containing all the well-formed numerical expressions for
the above number systems as well as error expressions such as, for example, 4 + 7/0. This
allows support for partial functions in a total setting, in the sense that a function whose appli-
cation to some arguments has a kind but not a sort should be considered undefined for those
arguments (but notice that functions can also map undefined to defined results, for example
in the context of error recovery). Furthermore, operators can be subsort-overloaded, providing
a useful form of subtype polymorphism. For example, the addition operation _+_ is subsort
overloaded and has typings for each of the above number sorts. A further feature, greatly
extending the expressive power for specifying partial functions, is the possibility of defining
sorts by means of equational conditions. For example, a sequential composition operation _;_
concatenating two paths in a graph is defined if and only if the target of the first path coincides
with the source of the second path. In Maude this can be easily expressed with the “conditional
membership” (see Section 4.3):
vars P Q : Path .
cmb (P ; Q) : Path if target(P) = source(Q) .
Generic types and modules
Maude supports a powerful form of generic programming that substantially extends the param-
eterized programming capabilities of OBJ3 [69]. The analogous terminology to express these
capabilitiesinhigher-ordertypetheorywouldbeparametric polymorphism anddependent types.
ButinMaudetheparametersarenotjusttypes, buttheories, includingoperatorsandequations
that impose semantic restrictions on the parameterized module instantiations. Thus, whereas a
parametric LIST module can be understood just at the level of the parametric type (sort) of list
elements, a parameterized SORTING module has the theory TOSET of totally ordered sets as its
parameter, including the axioms for the order predicate, that must be satisfied in each correct
instance for the sorting function to work properly. Types analogous to dependent types are
also supported by making the parameter instantiations depend on specific parametric constants
in the parameter theory, and by giving membership axioms depending on such constants. For
example, natural numbers modulo n (see Section 12.5), and arrays of length n, can be easily
defined this way.
Support for objects
The bank accounts example illustrates a general point, namely, that in Maude it is very easy
to support objects and distributed object interactions in a completely declarative style with
rewrite rules. Although such object systems are just a particular style of system modules
in which object interactions (through messages or directly between objects) are expressed by
rewriting, Maude provides special support for object-based programming and for fair execution
of object-based applications (see Chapter 6.4). Furthermore, Maude provides special syntax
in object-oriented modules (see Chapter 6). Such modules directly support object-oriented
concepts like objects, messages, classes, and multiple class inheritance. Moreover, the support
for communication with external objects (see Section 9) allows Maude objects to interact by
message passing with internet sockets and, through them, with all kinds of other external
objects, such as files, databases, graphical user interfaces, sensors, robots, and so on. All this
is achieved without compromising Maude’s declarative nature: interaction with normal Maude
objects and with external objects can both be programmed with rewrite rules. Using internet
sockets as external objects, it is also easy to develop distributed implementations in Maude,
where a “soup” of objects and messages is not realized just as a multiset data structure in a

<!-- manual page 19 -->
1.1. SIMPLICITY, EXPRESSIVENESS, AND PERFORMANCE 7
single sequential machine, but as a “distributed soup,” with objects and messages in different
machines or in transit.
Reflection
This is a very important feature of Maude. Intuitively, it means that Maude programs can
be metarepresented as data, which can then be manipulated and transformed by appropriate
functions. It also means that there is a systematic causal connection between Maude mod-
ules themselves and their metarepresentations, in the sense that we can either first perform
a computation in a module and then metarepresent its result, or, equivalently, we can first
metarepresent the module and its initial state and then perform the entire computation at the
metalevel. Finally, the metarepresentation process can itself be iterated giving rise to a very
useful reflective tower. Thanks to Maude’s logical semantics (more on this in Section 1.2), all
this is not just some kind of “glorified hacking,” but a precise form of logical reflection with a
well-defined semantics (see Chapter 17 and [28, 29]). There are many important applications
of reflection. Let us mention just three:
• Internal strategies. Since the rewrite rules of a system module can be highly nondeter-
ministic, there may be many possible ways in which they can be applied, leading to quite
different outcomes. In a distributed object system this may be just part of life: provided
some fairness assumptions are respected, any concurrent execution may be acceptable.
But what should be done in a sequential execution? Maude does indeed support two
different fair execution strategies in a built-in way through its rewrite and frewrite
commands (see Section 5.4). But what if we want to use a different strategy for a given
application? Maude provides two answers for this question, the fixed object-level strat-
egy language explained in Section 10, and the possibility of executing Maude modules
at the metalevel with user-definable internal strategies5 (see Section 17.7). Such internal
strategies can be defined by rewrite rules in a metalevel module that guides the possi-
bly nondeterministic application of the rules in the given “object level” module. This
process can be iterated in the reflective tower. That is, we can define meta-strategies,
meta-meta-strategies, and so on.
• Module algebra. The entire module algebra in which parameterized modules can be com-
posed and instantiated becomes expressible within the logic, and extensible by new mod-
ule operations that transform existing modules metarepresented as data. This is of more
than theoretical interest: Maude’s module algebra is realized exactly in this way by Full
Maude 6, a Maude program defining all the module operations and easily extensible with
new ones.
• Formal tools. The verification tools in Maude’s formal environment must take Maude
modules as arguments and perform different formal analyses and transformations on such
modules. This is again done by reflection in tools such as Maude’s inductive theorem
prover, the Church-Rosser checker, the Maude termination tool, the Real-Time Maude
tool, and so on.
1.1.3 Performance
Achieving expressiveness in all the ways described above without sacrificing performance is
a nontrivial matter. Successive Maude implementations have been advancing this goal while
expanding the set of language features. More work remains ahead, but it seems fair to say that
Maude, although still an interpreter, is a high-performance system that can be used for many
5That is, internal to Maude’s logic, in the sense of being definable by logical axioms.
6Full Maude is available at https://github.com/maude-team/full-maude.

<!-- manual page 20 -->
8 CHAPTER 1. INTRODUCTION
non-toyapplicationswithcompetitiveperformanceandwithmanyadvantagesoverconventional
code. Without in any way trying to extrapolate a specific experience into a general conclusion,
a concrete example from the Maude user’s trenches may illustrate the point. A formal tool
component to check whether a trace of events satisfies a given linear temporal logic (LTL)
formula was written in Maude at NASA Ames by Grigore Roşu in about one page of Maude
code. The component had a trivial correctness proof—the Maude module was based on the
equational definition of the LTL semantics for the different connectives. This replaced a similar
component having about 5,000 lines of Java code that had taken over a month to develop by an
experienced colleague. The Java tool used a translation of LTL formulas into Büchi automata
(the usual method to efficiently model check an LTL formula) and run about three times more
slowly than the Maude code. It would have been very difficult to prove the correctness of the
Java tool and, having a better and clearly correct alternative in the Maude implementation,
this was never done.
Generally and roughly speaking, the current Maude implementation can execute syntac-
tic rewriting with typical speeds from half a million to several million rewrites per second,
depending on the particular application and the given machine. Similarly, associative and
associative-commutative equational rewriting with term patterns used in practice7 can be per-
formed at the typical rate of one hundred thousand to several hundred thousand rewrites per
second.
These figures must be qualified by the observation that, until recently, the cost of an as-
sociative or associative-commutative rewriting step depended polynomially on the size of the
subject term, even with the most efficient algorithms. In practice this meant that this kind of
rewriting was not practical for large applications, in which the lists or multisets to be rewritten
could have millions of elements. This situation has been drastically altered by a result of Steven
Eker [51] providing new algorithms for associative and associative-commutative rewriting that,
for patterns typically encountered in practice, can perform one step of associative rewriting
in constant time, and one associative-commutative rewriting step in time proportional to the
logarithm of the subject term’s size. Maude supports equational rewriting with these new
algorithms.
The reason why the Maude interpreter achieves high performance is that the rewrite rules
are carefully analyzed and are then semicompiled into efficient matching and replacement au-
tomata [49] with efficient matching algorithms. One important advantage of semicompilation
is that it is possible to trace every single rewriting step. More performance is of course possible
by full compilation. Maude has an experimental compiler for a subset of the language which
can typically achieve a fivefold speedup over the interpreter.
Four other language features give the user different ways of optimizing the performance of
his/her code. One is profiling, allowing a detailed analysis of which statements are most expen-
sive to execute in a given application (see Section 21.1.5). Another is evaluation strategies (see
Section 4.4.7), giving the user the possibility of indicating which arguments and in which order
to evaluate before simplifying a given operator with the equations. This can range from “no
arguments” (a lazy strategy) to “all arguments” (an eager bottom-up strategy) to something in
the middle (like evaluating the condition before simplifying an if-then-else expression). Evalu-
ation strategies control the positions in which equations can be applied. But what about rules?
The analogous feature for rules is that of frozen argument positions; that is, declaring certain
argument positions in an operator with the frozen attribute (see Section 4.4.9) blocks rule
rewriting anywhere in the subterms at those positions. A fourth useful feature is memoization
7In its fullest generality, it is well known that associative-commutative rewriting is an NP-complete problem.
In programming practice, however, the patterns used as lefthand sides allow much more efficient matching,
so that the theoretical limits only apply to “pathological” patterns not encountered in typical programming
practice.

<!-- manual page 21 -->
1.2. THE LOGICAL FOUNDATIONS OF MAUDE 9
(see Section 4.4.8). By giving an operator the memo attribute, Maude stores previous results
of function calls to that symbol. This allows trading off space for time, and can lead in some
cases to drastic performance improvements.
One nagging question may be reflection. Is reflection really practical from a performance
perspective? The answer is yes. In Maude, reflective computations are performed by descent
functions that move metalevel computations to the object level whenever possible (see Sec-
tion 17.6). This, together with the use of caching techniques, makes metalevel computations
quiteefficient. Atypicalmetalevelcomputationmayperformmillionsofrewritesveryefficiently
at the object level, paying a cost (linear in the size of the term) in changes of representation
from the metalevel to the object level and back only at the beginning and at the end of the
computation.
1.2 The logical foundations of Maude
The foundations of a house do not have to be inspected every day: one is grateful that they are
there and are sound. This section describes the logical foundations of Maude in an informal,
impressionistic style, not assuming much beyond a cocktail party acquaintance with logic and
mathematics. The contents of this section may be read in two ways, and at two different
moments:
• before reading the rest of the manual, to obtain a bird’s-eye view of the mathematical
ideas underlying Maude’s design and semantics; or
• afterreadingtherestofthemanual, togainamoreunifiedunderstandingofthelanguage’s
design philosophy and its foundations.
Readers with a more pragmatic interest may safely skip this section, but they may miss
some of the fun.
Maude is a declarative language in the strict sense of the word. That is, a Maude program
is a logical theory, and a Maude computation is logical deduction using the axioms specified in
the theory/program. But which logic? There are two, one contained in the other. The seamless
integration of the functional world within the broader context of concurrent, nondeterministic
computation is achieved at the language level by the inclusion of functional modules as a special
case of system modules. At the mathematical level this inclusion is precisely the sublogic
inclusion in which membership equational logic [95, 14] is embedded in rewriting logic [91, 16].
A functional module specifies a theory in membership equational logic. Mathematically, we
can view such a theory as a pair (Σ,E∪A). Σ, called the signature, specifies the type structure:
sorts, subsorts, kinds, and overloaded operators. E is the collection of (possibly conditional)
equations and memberships declared in the functional module, and Ais the collection of equa-
tional attributes (assoc, comm, and so on) declared for the different operators. Computation
is of course the efficient form of equational deduction in which equations are used from left to
right as simplification rules.
Similarly, a system module specifies a rewrite theory, that is, a theory in rewriting logic.
Mathematically, such a rewrite theory is a 4-tuple R= (Σ,E∪A,ϕ,R), where (Σ,E∪A) is
the module’s equational theory part, ϕ is the function specifying the frozen arguments of each
operator in Σ, and R is a collection of (possibly conditional) rewrite rules. Computation is
rewritinglogicdeduction, inwhichequationalsimplificationwiththeaxiomsE∪Aisintermixed
with rewriting computation with the rules R.
We can of course view an equational theory (Σ,E∪A) as a degenerate rewrite theory of
the form (Σ,E ∪A,ϕ∅,∅), where ϕ∅(f) = ∅, that is, no argument of f is frozen, for each
operator f in the signature Σ. This defines a sublogic inclusion from membership equational

<!-- manual page 22 -->
10 CHAPTER 1. INTRODUCTION
logic (MEqLogic) into rewriting logic (RWLogic) which we can denote
MEqLogic →RWLogic.
In Maude this corresponds to the inclusion of functional modules into the broader class of
system modules. However, Maude’s inclusion is more general: the user can give the desired
freezing information for each operator in the signature of a functional module, not just the ϕ∅
above.
Another important fact is that each Maude module specifies not just a theory, but also an
intended mathematical model. This is the model the user has intuitively in mind when writing
the module. For functional modules such models consist of certain sets of data and certain
functions defined on such data, and are called algebras. For example, the intended model for
a NAT module is the natural numbers with the standard arithmetic operations. Similarly, a
module LIST-QID may specify a data type of lists of quoted identifiers, and may import NAT
and BOOL as submodules to specify functions such as length and _in_. Mathematically, the
intended model of a functional module specifying an equational theory (Σ,E∪A), with Σ the
signature defining the sorts, subsorts, and operators, E the equations and memberships, and
A the equational attributes like assoc, comm, and so on, is called the initial algebra of such a
theory and is denoted TΣ/E∪A.
In a similar way, a system module specifying a rewrite theory R= (Σ,E∪A,ϕ,R) has an
initial model, denoted TR, which in essence is an algebraic (labeled) transition system.
8 The
states and data of this system are elements of the underlying initial algebra TΣ/E∪A. The state
transitions are the (possibly complex) concurrent rewrites possible in the system by application
of the rules R. For our bank accounts example, these transitions correspond to all the possible
concurrent computations that can transform a given “soup” of account objects and messages
into another soup. Again, this is the model the programmer of such a system has in mind.
How do the mathematical models associated with Maude modules and the computations
performed by them fit together? Very well, thanks. This is the so-called agreement between the
mathematical semantics (themodels)andtheoperational semantics (thecomputations). Inthis
introduction we must necessarily be brief; see Sections 4.6 and 4.7 and [14] for the whole story in
the case of functional modules, and Section 5.3 and [116] for the case of system modules. Here is
the key idea: under certain executability conditions required of Maude modules, both semantics
coincide. For functional modules we have already mentioned that the equations should have
good properties as simplification rules, so that they evaluate each expression to a single final
result. Technically, these are called the Church-Rosser and termination assumptions. Under
theseassumptions, thefinalvalues, calledthecanonical forms, ofallexpressionsformanalgebra
called the canonical term algebra. By definition, the results of operations in this algebra are
exactly those given by the Maude interpreter: this is as computational a model as one can
possibly get. For example, the results in the canonical term algebra of the operations
length(’a . ’b . ’c . nil)
’b in (’a . ’b . ’c . nil)
are, respectively,
s s s 0
true
Suppose that a functional module specifies an equational theory (Σ,E∪A) and satisfies the
Church-Rosser and termination assumptions. Let us then denote by CanΣ/E∪A the associated
8With additional operations, including a sequential composition operation for labeled transitions.

<!-- manual page 23 -->
1.3. PROGRAMMING, SPECIFICATION, AND VERIFICATION 11
canonical term algebra. The coincidence of the mathematical and operational semantics is then
expressed by the fact that we have an isomorphism
TΣ/E∪A
∼
= CanΣ/E∪A.
In other words, except for a change of representation, both algebras are identical.
For system modules, the executability conditions center around the notion of coherence
between rules and equations (see [116] and Section 5.3). The equational part E∪A should be
Church-Rosser and terminating as before. A reasonable strategy (the one adopted in Maude
by the rewrite command, see Chapter 5) is to first apply the equations to reach a canonical
form, and then do a rewriting step with a rule in R. But is this strategy complete? Couldn’t we
miss rewrites with Rthat could have been performed if we had not insisted on first simplifying
the term to its canonical form with the equations? Coherence guarantees that this kind of
incompleteness cannot happen (see Section 5.3).
1.3 Programming, specification, and verification
The observations in the previous section about the agreement between mathematical and op-
erational semantics in Maude programs are of enormous importance for reasoning about them
and verifying their correctness. The key point is that there are three different uses of Maude
modules:
1. As programs, to solve some application. In principle we could have programmed such
an application in some other programming language, but we may have chosen Maude
because its features make the programming task easier and simpler.
2. As formal executable specifications, that provide a rigorous mathematical model of an
algorithm, a system, a language, or a formalism. Because of the agreement between
operational and mathematical semantics, this mathematical model is at the same time
executable. Therefore, we can use it as a precise prototype of our system to simulate
its behavior. The system itself could be implemented in a conventional language, or
perhaps in Maude itself (as in (1) above) as a more detailed Maude program, or maybe
our specification is already detailed and efficient enough to be directly used as its own
implementation.
3. As models that can be formally analyzed and verified with respect to different properties
expressing various formal requirements. For example, we may want to prove that our
Maude module terminates; or that its equations have the Church-Rosser property; or that
a given function, equationally defined in the module, satisfies some properties expressed
as first-order formulas. Similarly, given a system module we may want to model check
some properties about it, such as the satisfaction of some invariants or, more generally,
of some temporal logic formulas.
Note that the distinction between uses (1) and (2) is, for the most part, in the eyes of
the beholder. In fact, there is a seamless integration of specifications and code. The same
Maude module can simultaneously be viewed as an executable formal specification and as a
program. Furthermore, certain kinds of formal requirements needed for verification in (3) can
be expressed at the Maude level, either in Maude theories (see Section 7.4.1), or by including
some nonexecutable statements in a Maude module giving them the nonexec attribute (see
Section 4.5.3). This can be very useful in several ways. For example, we may include lemmas
that we have proved about a module, either in theories or as nonexecutable statements in the
module itself. Similarly, we may begin with some nonexecutable specifications in a Maude
theory, and then refine them using views (see Section 7.4.2) into the desired Maude module
satisfying them.

<!-- manual page 24 -->
12 CHAPTER 1. INTRODUCTION
There is, however, no need for all the properties that we wish to formally verify in (3) to be
in the logic of Maude, that is, to be statements in membership equational logic or in rewriting
logic. More generally, properties can be expressed, for example, as arbitrary first-order logic
formulas, or as temporal logic formulas. An interesting issue is then to explain precisely what
it means for a Maude module, defined in membership equational logic or in rewriting logic,
to satisfy a formula in one of those logics. Here is where the Maude initial model semantics
explained in Section 1.2 becomes crucial. Such a semantics means that what a Maude module
denotes is a specific mathematical model, namely, the initial one. Satisfaction of any property,
expressed as some kind of formula, means satisfaction of that formula in the initial model. This
is an important observation, even when the formula in question is expressed in Maude’s native
logic. Let us explain this idea in more detail.
Consider, for example, that we have defined natural number addition in a Maude functional
module with Peano notation, so that zero is represented as the constant 0, and there is a
successor function s_ so that, for example, 2 is represented as s s 0. Natural number addition
can then be defined by the equations
op _+_ : Nat Nat -> Nat .
vars N M K : Nat .
eq N + 0 = N .
eq N + (s M) = s (N + M) .
The initial model of these equations is precisely the algebra of the natural numbers with
zero, successor, and the usual addition function. For example, using the canonical term algebra
representation (see Section 1.2), when we add s s 0 and s s 0 in this algebra we obtain the
result s s s s 0.
Consider now two relevant properties of natural number addition, namely, associativity and
commutativity. These properties are precisely described by the respective equations
eq N + M = M + N [nonexec] .
eq N + (M + K) = (N + M) + K [nonexec] .
where we have used the nonexec attribute to emphasize that these equations are not part of
our natural number addition module, and are not meant to be executed (in fact, if executed
the first equation would loop). They may, for example, be stated in a separate Maude theory
as properties we wish to verify.
The first thing to observe is that the above associativity and commutativity equations are
not provable by equational deduction, that is, they do not follow by replacing equals by equals
from the two equations defining the addition function. They are in fact inductive properties of
the addition function. Therefore, in order to prove them, using for example Maude’s inductive
theorem prover (ITP), we need to use a stronger proof method, namely, Peano induction. But
for any equational specification, being an inductive property and being a property satisfied by
its initial model are one and the same thing [98]. Therefore, what we mean when we say that
our natural number addition module satisfies the associativity and commutativity equations is
precisely that its initial model does.
Of course, associativity and commutativity are properties expressible in Maude’s native
logic (in fact, just in its equational sublogic). But the case of arbitrary first-order formulas is
entirely similar. Consider, for example, the property that any even number is the sum of two
odd numbers, which can be expressed as the first-order formula
∀n: Nat (even(n) =⇒ ∃x,y: Nat (odd(x) ∧odd(y) ∧n= x+ y)).
Let us assume, for argument’s sake, that we had also defined the odd and even predicates
in our Maude natural number module. What does it mean for our module to satisfy the

<!-- manual page 25 -->
1.3. PROGRAMMING, SPECIFICATION, AND VERIFICATION 13
above formula? Just as before, it exactly means that the initial model denoted by our Maude
specification satisfies the formula. The point is that membership equational logic is a sublogic
of many-kinded first-order logic with equality (MKFOL=) that we can represent with a sublogic
inclusion
MEqLogic →MKFOL=
.
Therefore, our initial model is also a first-order logic model, and it is perfectly clear what it
means for it to satisfy a first-order formula.
In a similar way, if we have a Maude system module and choose an initial state for it, we may
be interested in verifying that it satisfies a given temporal logic formula. Defining satisfaction in
this case is not as direct as for first-order formulas, because we do not have a sublogic inclusion
from rewriting logic into temporal logic. However, the meaning of satisfaction in this case is
also fairly straightforward. The point is that to such a system module, that is, to a rewrite
theory in which we have defined some atomic state predicates equationally, we can naturally
associate a Kripke structure (see Section 12.2). Since Kripke structures are the standard models
of temporal logic, satisfaction of the given temporal logic formula exactly means that the Kripke
structure associated to the module satisfies the formula. In fact, such a Kripke structure and
the initial model of the rewrite theory are intimately related, so that the initial model can be
used to define the corresponding Kripke structure. As explained in Chapters 11 and 12, if our
system module is such that the set of states reachable from the initial state is finite, we can
use Maude’s search command and Maude’s model checker for linear temporal logic (LTL) as
decision procedures to verify, respectively, the satisfaction of invariants and of LTL properties.
Besides being able to use Maude’s inductive theorem prover (ITP) to verify inductive prop-
erties of functional modules, and the above-mentioned built-in support for verifying invariants
and LTL formulas through the search command and Maude’s LTL model checker, we can use
the following Maude tools to formally verify other properties:
• the Maude Termination Tool (MTT) [39, 38, 47, 48] can be used to prove termination of
functional modules (see Section 11.4);
• the Maude Church-Rosser Checker (CRC) [43, 45, 46, 25, 40, 47, 48] can be used to check
the Church-Rosser property of unconditional functional modules (see Section 11.4);
• the Maude Coherence Checker (ChC) [44, 45, 47, 48] can be used to check the coherence
(or ground coherence) of unconditional system modules (see Section 11.4); and
• the Maude Sufficient Completeness Checker (SCC) [73, 47, 48] can be used to check
that defined functions have been fully defined in terms of constructors (see Sections 4.4.3
and 11.4).
These tools are integrated in what is called the Maude Formal Environment (MFE), available
at https://github.com/maude-team/MFE. Furthermore, if we are dealing with rewriting logic
specifications of real-time and hybrid systems, we can use the Real-Time Maude tool to both
simulate such specifications and to perform search and model-checking analysis of their LTL
properties [106, 107]. You can find information on how to use each of these tools in their
corresponding web sites.
In summary, therefore, Maude supports three seamlessly integrated tasks: programming,
executable formal specification, and formal analysis and verification. For analysis and verifi-
cation purposes, the Maude interpreter itself is the first and most obvious tool. It is in fact a
high-performance logical engine that can be used to prove certain kinds of logical facts about
our theories. For example, we can use the Maude interpreter as a decision procedure for equa-
tional deduction if the desired theory has good properties. Similarly, as already mentioned, we
can use it also to verify invariants and LTL properties of finite-state system modules. More
generally, we can use other tools in Maude’s formal environment, such as the ITP, MTT, CRC,
ChC, and SCC tools (or Real-Time Maude for real-time systems) to formally verify a variety

<!-- manual page 26 -->
14 CHAPTER 1. INTRODUCTION
of other properties.
1.4 A high-performance logical framework
Our previous discussion of the programming, executable specification, and formal verification
uses of Maude makes clear that we can distinguish two different levels of formal specification:
a system specification level, and a property specification one. In a system specification we are
after an unambiguous specification of a given system and how it actually works. Ideally this
specification should be both formal and executable, and should therefore provide an executable
mathematical model of the system we are interested in. This is exactly what Maude modules
provide.
Bycontrast,whenspecifyingproperties ofasystemwearenotnecessarilyafteranexecutable
model of oursystem. Instead, we assume it, aseitheralready given or to be developed later, and
specify such properties in a typically nonexecutable manner: for example in first-order logic,
higher-order logic, or some temporal logic. That is, the properties we specify have an intended
model, namely the system design captured by a system specification, and we are interested in
verifying by different methods that the intended model satisfies the properties stated in our
property specification. In the context of Maude, such property specifications can be given in a
variety of ways:
• as nonexecutable equations, memberships, and rules in Maude’s native logics;
• as first-order logic formulas; or
• as invariants or, more generally, linear temporal logic formulas.
We can then use Maude itself and its formal tool environment to try to verify that a given
system specified as a Maude module satisfies the desired properties.
Since Maude system specifications should be both formal and executable, Maude native
logics, namely, membership equational logic and its rewriting logic extension, should be com-
putational logics, that is, logics in which computation and deduction coincides, and simple
enough to allow a high-performance implementation as a declarative programming language.
This is what the Maude implementation provides. Of course, as mentioned in Section 1.2 and
further explained in Sections 4.6 and 5.3, Maude modules should be theories that satisfy some
reasonable executability requirements, making possible not only their efficient execution, but
also the already-mentioned coincidence between mathematical and operational semantics.
However, not all computational logics are equally expressive. For example, equational log-
ics (in either first-order or higher-order versions) are very well suited to specify deterministic
systems under the Church-Rosser assumption, but poorly equipped to specify concurrent and
highly nondeterministic systems. The whole point of extending membership equational logic
to rewriting logic is to seamlessly integrate the specification of deterministic systems, through
equational specifications in functional modules, and of concurrent and nondeterministic sys-
tems, through rewriting logic specifications in system modules, within the same language.
Experience has shown that this makes rewriting logic a very expressive semantic framework for
system specification. Here we only mention some relevant areas:
• Models of computation. Many models of computation, including a very wide range of
concurrency models, can be naturally specified as different theories within rewriting logic,
and can be executed and analyzed in Maude.
• Programming languages. Rewriting logic has very good properties—combining in a sense
the best features of denotational semantics’ equational definitions with the strengths of
structural operational semantics—to give formal semantics to a programming language.
Furthermore, in Maude such semantics definitions become the basis of interpreters, model
checkers, and other program analysis tools for the language in question.

<!-- manual page 27 -->
1.5. MANUAL STRUCTURE 15
• Distributed algorithms and systems. Because of its good features for concurrent, object-
based specification, many distributed algorithms and systems, including, for example,
network protocols and cryptographic protocols, can be easily specified and analyzed in
Maude. Furthermore, making use of Maude’s external object facility to program interac-
tions with internet sockets, one can not just specify but also program various distributed
applications in a declarative way (see Section 9).
• Biological systems. Cell dynamics is intrinsically concurrent, since many different bio-
chemical reactions happen concurrently in a cell. By modeling such biochemical reactions
with rewrite rules, one can develop useful symbolic mathematical models of cell biology.
Such models can then be used to study and predict biological phenomena.
Furthermore, other application areas can be naturally supported in appropriate extensions
of rewriting logic and Maude. For example, real-time and hybrid systems can be specified as
real-time rewrite theories. Such specification can be executed and analyzed in the Real-Time
Maude tool [106, 107]. Similarly, probabilistic systems can be specified as probabilistic rewrite
theories, and can be simulated in PMaude and analyzed in the VeStA tool [79, 2].
The fact that in a computational logic computation and deduction coincide, so that they
are like two sides of the same coin, can be used in two ways: we can use the logic as a semantic
framework to specify different computational entities as just explained; or we can use it as a
logical framework to represent many other logics in it. That is, if our computational logic has
good representational features, it can be used as a universal logic which can faithfully express
the inference systems of many other logics.
Since the logic is computational and presumably has an efficient implementation, this is not
just a purely theoretical exercise: we can use such an implementation to mechanize deduction
in any logic that we can faithfully represent inside our logical framework. Experience has shown
that rewriting logic has very good properties as a logical framework in precisely this sense. An
important practical consequence is that it becomes quite easy to use Maude to develop a variety
of formal tools for different logics. The point is that any such tool has an associated inference
system, so it is just a matter of representing such an inference system as a rewrite theory and
guiding the application of the inference rules with suitable strategies (see Section 17.7). In
addition, since such formal tools often manipulate and transform not only formulas but also
theories, Maude’s reflective capabilities, which allow manipulating theories as data, become
enormously useful [25, 26].
Reflection and the existence of initial models (and therefore of induction principles for such
models) have one further important consequence, namely, that rewriting logic has also good
properties as a metalogical framework. A metalogical framework is a logical framework in
which we can not only represent and simulate many other logics: we can also reason within
the framework about the metalogical properties of the logics thus represented. As explained
in [8], this is exactly what can be done in rewriting logic using Maude and Maude’s inductive
theorem prover (ITP).
1.5 Manual structure
The present manual documents Maude 3.3.1, and explains Maude’s basic concepts in a leisurely
and mostly informal style. The material is basically presented following a “grammatical” order;
for example, all features related with operators are discussed together. Concepts are introduced
by concrete examples, that may be fragments of modules. The complete module examples are
available in the official webpage https://maude.cs.illinois.edu. We follow the convention
of naming each file in that site as the module it contains, or if a file contains several modules, as
the main one. In most cases, the rewriting or search commands and the corresponding outputs

<!-- manual page 28 -->
16 CHAPTER 1. INTRODUCTION
are also included in the files.
Here is a brief summary of what can be found in the remaining chapters of this manual:
Chapter 2 explains how to get Maude, how to install the system on the different platforms
supported, and how to run it. It also includes pointers on how to get additional informa-
tion and support.
Chapter 3 describes the basic syntactic constructs of the language, including what is an iden-
tifier, a sort, and an operator. The different kinds of declarations that can be included
in the different types of modules are explained here, in addition to fundamental concepts
such as kinds or terms, and a discussion on parsing.
Chapter 4 introduces functional modules, and the different statements that can be found in
this kind of modules, namely equations and membership axioms. Operator and statement
attributes are also introduced. The final part of this chapter is devoted to the use of
functional modules for equational simplification, for which matching modulo axioms is a
fundamental feature.
Chapter 5 introduces system modules, and is mainly devoted to rules, term rewriting, and
the search command.
Chapter 6 introduces object-oriented modules, which provide a syntax more convenient than
that of system modules for object-oriented applications, with direct support for the dec-
laration of classes, inheritance, and useful default conventions in the definition of rules.
However, object-oriented modules are internally desugared into system modules. This
chapter also explains the basic support for object-based programming, with special em-
phasis on the standard notation for object systems, and the transformation carried out.
Chapter 7 explains the support for modularity provided by Maude. It describes first the
different modes of module importation, namely protecting, extending, generated-by, and
including. Then it introduces the module summation and renaming operations. Finally,
thischapterexplainsthepowerfulformofparameterizedprogrammingavailableinMaude,
based on theories and views.
Chapter 8 provides detailed descriptions of the different predefined data types available, in-
cluding Booleans, natural numbers, integers, rationals, floating-point numbers, strings,
andquotedidentifiers. ItalsodescribesthegenericcontainersprovidedbyMaude, namely
lists, sets, maps, and arrays. The chapter finishes with a description of a built-in linear
Diophantine equation solver.
Chapter 9 explains Maude’s support for rewriting with external objects and the implemen-
tation of sockets, standard streams, and files. Facilities to manage system directories,
processes and machine time are also implemented as external objects.
Chapter 10 describes Maude’s object-level strategy language and the corresponding strategy
modules. It contains many examples of using strategies to control rewriting.
Chapter 11 explains how to use the search command to model check invariant properties of
concurrent systems specified as system modules in Maude.
Chapter 12 introduces linear temporal logic (LTL) and describes the facilities for LTL model
checking provided by the Maude system. This procedure can be used to prove properties
when the set of states reachable from an initial state in a system module is finite. When
this is not the case, it may be possible to use an equational abstraction technique for
reducing the size of the state space.
Chapter 13 describes Maude’s support of order-sorted unification modulo axioms such as
either commutativity or associativity and commutativity. The importance of this feature
is made explicit in an overview of several interesting applications of unification, including
narrowing and symbolic reachability analysis. This chapter also includes a discussion on
endogenous vs. exogenous order-sorted unification algorithms.

<!-- manual page 29 -->
1.6. THE MAUDE BOOK 17
Chapter 14 introduces the concept of variants, and then explains its use in the get variants
command and its application in variant-based equational order-sorted unification.
Chapter 15 describes the implementation of narrowing based on the unification and variant
facilities introduced in the previous Chapters 13 and 14.
Chapter 16 introduces satisfiability modulo theories (SMT) and describes the connection of
Maude to SMT solvers like CVC4 and Yices2.
Chapter 17 presents the reflective capabilities of the Maude system. The concept of reflection
isintroduced, andtheeffectivewayofsupportingmetalevelcomputationisdiscussed. The
predefined module META-LEVEL and its submodules are presented, with special emphasis
on the descent functions provided. The chapter ends with an introduction to the notion
of internal strategies.
Chapter 18 explains the way of using the facilities provided by the modules META-LEVEL,
STD-STREAM, and LEXICAL for the construction of user interfaces and metalanguage ap-
plications.
Chapter 19 introduces meta-interpreters. Each meta-interpreter is an external object that is
an independent Maude interpreter, complete with module and view databases, and able
to send and receive messages. Together with standard streams, meta-interpreters can be
used to develop execution environments.
Chapter 21 discusses debugging and troubleshooting, considering the different debugging fa-
cilities provided: tracing, term coloring, the debugger, and the profiler. A number of
traps and known problems are also commented.
Chapter 20 describes the support for the generation of LATEX code from the interaction with
the interpreter.
Appendix A gives a complete list of the commands available in Maude.
Appendix B includes the grammar of Maude.
1.6 The Maude book
Most of the material in this manual also appears in the book All About Maude: A High-
Performance Logical Framework, published by Springer as volume 4350 in the series Lecture
Notes in Computer Science [24].
The book contains many additional examples and explanations, as well as information on
applications and tools.
We gratefully acknowledge the permission given by Springer to distribute this manual on
the web.
Acknowledgements
Languages are living organisms. The lifeblood provided by experienced users is key to their
growthandtheirimprovement. Wehavebenefitedmuchfromcolleagueswhohaveuseddifferent
alpha versions of Maude; we cannot mention them all, but Luis Aguirre, Christiano Braga, Feng
Chen, GritDenker, AzadehFarzan, JoeHendrix, MerrillKnapp, NirmanKumar, ÓscarMartín,
Miguel Palomino, Peter Ölveczky, José Quesada, Adrián Riesco, Dilia Rodriguez, Grigore Roşu,
Ralf Sasse, Koushik Sen, Ambarish Sridharanarayanan, Mark-Oliver Stehr, Prasanna Thati,
andAlbertoVerdejodeservespecialthanksfortheircreativeusesofMaudeandtheirsuggestions
for improving the language. Thanks to Christiano Braga, Peter Mosses, Peter Ölveczky, Miguel
Palomino, Sylvan Pinsky, Isabel Pita, Adrián Riesco, Dilia Rodriguez, Manuel Roldán, Julia
Sapiña, Mark-Oliver Stehr, Antonio Vallecillo, and Alberto Verdejo for their comments on
previous versions of this document.

<!-- manual page 30 -->
18 CHAPTER 1. INTRODUCTION
As already mentioned, Maude’s historical precursor is the OBJ3 language [69]. The OBJ3
experience has greatly influenced the Maude design and philosophy, and we are grateful to all
our former OBJ colleagues for this. Joseph Goguen should be mentioned in particular, because
of his enormous influence in all aspects of OBJ; and Tim Winkler for having implemented a
state-of-the-art OBJ3 system with such great skill.
Two other rewriting logic languages, ELAN [11] and CafeOBJ [65], have provided a rich
stimulus to the design of Maude. Although our language design solutions have often been dif-
ferent, we have all been wrestling with a similar problem: how to best obtain efficient language
implementations of rewriting-based languages. We have benefited much from the ELAN and
CafeOBJ experience, and from many discussions with their main designers and implementers:
ClaudeandHélèneKirchner, MarianVittek, Pierre-EtienneMoreau, KokichiFutatsugi, Râzvan
Diaconescu, Ataru Nakagawa, Toshimi Sawada, and Makoto Ishisone.
Bringing a new language design to maturity requires a long-term research effort and sub-
stantial resources. We are not there yet, but much has been advanced since the early design
phases. Perhaps the longest, most sustained support has come from the US Office for Naval
Research (ONR) through a series of contracts. We are most grateful to Dr. Ralph Wachter at
ONR for his continued encouragement at every step of the way. The US Defense Advance Re-
search Projects Agency (DARPA), the US National Science Foundation (NSF), and the Spanish
Ministry for Education and Science (MEC) have also contributed important resources to the
development of Maude, its foundations, and its applications.

