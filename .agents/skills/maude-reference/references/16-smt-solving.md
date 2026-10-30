# Chapter 16: SMT Solving (manual pages 425-430)

<!-- manual page 425 -->
Chapter 16
SMT Solving
Given a Σ-algebra A and a quantifier-free (QF) formula φ with variables Y, we say that φ is
satisfiable (resp. valid) in A iff there is an assignment map a : Y →A such that A,a |= φ
(resp. for all maps a : Y →A, A,a |= φ). Note that φ is valid in A iff¬φ is unsatisfiable in
A. We say that satisfiability in Ais decidable iff we have an algorithm such that given any QF
Σ-formula φit can answer whether or not φis satisfiable in A. Of course, this also means that
it can answer whether or not φ is valid in A.
Assuming that all sorts in Σ are nonempty, i.e., TΣ,s ̸= ∅for each sort s, we can view the
equational unification methods for theories (Σ,G) explained in Chapters 13 and 14, where G
can be either a set of axioms Ax, or a set of FVP equations E∪Ax, as providing a decision
procedure for satisfiability in the initial algebra TΣ/G for any positive (i.e., having no negations)
QF Σ-formula φ. This is so because any such positive φcan be put in disjunctive normal form
as φ≡ i∈I Qi, with each Qi the conjunction of a set of equations Qi. Then φis satisfiable
in TΣ/G iff there exists a j ∈I such that Qj has at least one G-unifier, which we can decide
in Maude by asking for the first unifier of the system of equations Qj.
In this chapter we broaden the picture about satisfiability begun in Chapters 13 and 14 by
explaining how satisfiability of QF formulas for several concrete Σ-algebras of interest is directly
supported in Maude by special modules and an interface of commands that makes accessible to
Maude users part of the functionality of an SMT solver. The reader should note that a given
Maude binary can only be connected to one SMT solver. The binaries distributed in the official
website are linked with Yices21, but instructions on how to compile Maude with CVC42 are
available in the compilation guide that comes with the Maude sources.
The most basic type of satisfiability is that of formulas in propositional logic, that is, of
Boolean expressions, in the two-element Boolean algebra B. This is called the SAT problem,
and decision procedures solving it are called SAT solvers. By definition, a Boolean expression
β with propositional variables X is satisfiable in B iff there is an assignment a : X →B of
Boolean values for the propositional variables X of β such that β is true under the assignment
a.
But note that such a propositional SAT-solving problem for β in B is equivalent to the
first-order satisfiability in the initial algebra B of the equational theory of the Booleans—as
specified, for example, in the BOOL-OPS functional module of Chapter 7.1—of the equation
β= true. In fact, Maude provides three different methods for solving the SAT problem: (1)
the BOOL-OPSmodule can decide if βis satisfiable in B: it is so iff its canonical form in BOOL-OPS
1Available at https://yices.csl.sri.com.
2Available at https://cvc4.github.io.
413

<!-- manual page 426 -->
414 CHAPTER 16. SMT SOLVING
is different from false;3 (2) the LTL SAT solver described in Section 12.4, since propositional
formulas are a special case of LTL formulas; and (3) the connection to a SAT solver explained
in this chapter, which is of course the most efficient method among those three.
Satisfiability modulo theories (SMT) solvers provide satisfiability procedures both for SAT-
solving and for various first-order theories T of interest. The theories discussed in this chapter
are all theories T associated to specific data types of interest in the following, precise sense:
given a Σ-algebra Aits first-order theory th(A) is the theory th(A) = (Σ,Γ(A)), where Γ(A) =
{φ|A|= φ}. Specifically, we will consider satisfiability procedures for QF formulas in theories
of the form th(A) of A: (i) Presburger arithmetic on the integers (Z,0,1,+,>,≥), (ii) linear
arithmetic on the rationals (Q,Q,+,>,≥), where Q is the (infinite) set of rational numbers
interpreted by themselves as constants of the algebra Q;4 and (iii) linear arithmetic for both the
integers and the rationals, corresponding, essentially, to a subsort inclusion Z ⊂Q. Therefore,
the intended model in this last case is in essence the linear arithmetic fragment of the initial
algebra of the theory RAT of rationals described in Section 7.6, which contains the sort Int
as a subsort. SMT solvers are called this way because they exploit the Boolean structure of
QF first-order formulas to perform SAT solving at the Boolean level in order to achieve more
efficient versions of satisfiability procedures for the theories T supported by the solver, using
the so-called DPLL(T) approach [101].
To make SMT solving for the above-described theories available to Maude users, we have
integrated within Maude, and have provided access to some of the decision procedures and
functionality of Yices21. Maude’s connection to Yices2 relies on the standard SMT-LIB no-
tation5 accepted by SMT solvers and allows for more connections to other SMT solvers to be
added in the future. As already mentioned, the specific theories currently supported in Maude
are: (i) SAT-solving for the Booleans, (ii) Presburger arithmetic for the integers, (iii) linear
arithmetic for the rationals; and (iv) linear arithmetic for both the integers and the rationals.
We describe below the corresponding Maude modules allowing definition of QF formulas in
those four theories.
16.1 Boolean formulas
The SMT interface is accessed by loading the file smt.maude. The SMT-LIB Core theory is
represented by the following signature.
fmod BOOLEAN is
sort Boolean .
op true : -> Boolean [special (id-hook SMT_Symbol (true))] .
op false : -> Boolean [special (id-hook SMT_Symbol (false))] .
op not_ : Boolean -> Boolean
[prec 53 special (id-hook SMT_Symbol (not))] .
op _and_ : Boolean Boolean -> Boolean
3See [110] for several alternative equational specifications besides BOOL-OPS, as well as a detailed explanation
of how satisfying assignments for satisfiable expressions can be obtained in an equational manner.
4This theory is the same as that of linear arithmetic with rational coefficients for the real numbers. That
is, for the algebra (R,Q,+,>,≥). The key observation is that any system of linear equations with rational
coefficients has a solution in the real numbers if and only if it has a solution in the rational numbers. The real
numbers are relevant only in the case of non-linear equations not supported here. Therefore, in what follows
we will not make any distinction between these two identical linear arithmetic theories and will just talk about
“linear arithmetic on the rationals.” Likewise, the theory’s main sort can be called either Rat or Real, since this
makes no difference. Actually, we will use Real to avoid confusion with the already existing sort Rat in the RAT
module.
5See https://smtlib.cs.uiowa.edu

<!-- manual page 427 -->
16.2. FORMULAS USING INTEGER LINEAR ARITHMETIC 415
[gather (E e) prec 55 special (id-hook SMT_Symbol (and))] .
op _xor_ : Boolean Boolean -> Boolean
[gather (E e) prec 57 special (id-hook SMT_Symbol (xor))] .
op _or_ : Boolean Boolean -> Boolean
[gather (E e) prec 59 special (id-hook SMT_Symbol (or))] .
op _implies_ : Boolean Boolean -> Boolean
[gather (e E) prec 61 special (id-hook SMT_Symbol (implies))] .
op _===_ : Boolean Boolean -> Boolean
[gather (e E) prec 51 special (id-hook SMT_Symbol (===))] .
op _=/==_ : Boolean Boolean -> Boolean
[gather (e E) prec 51 special (id-hook SMT_Symbol (=/==))] .
op _?_:_ : Boolean Boolean Boolean -> Boolean
[gather (e e e) prec 71 special (id-hook SMT_Symbol (ite))] .
endfm
Here a different sort Boolean and constants true and false are used for true and false to
avoid immediately ambiguity with Maude’s Bool sort and its constants true and false. All
of the operators are inert at the Maude level, i.e., they have no built-in semantics in Maude.
The _?_:_ operator is the if-then-else operation and _===_ and _=/==_ are used to represent
equality and inequality, again to minimize syntactic ambiguity with Maude’s standard symbols.
16.2 Formulas using integer linear arithmetic
The SMT-LIB Ints theory is represented by the following signature.
fmod INTEGER is
protecting BOOLEAN .
sort Integer .
op <Integers> : -> Integer [special (id-hook SMT_NumberSymbol (integers))] .
op -_ : Integer -> Integer
[special (id-hook SMT_Symbol (-))] .
op _+_ : Integer Integer -> Integer
[gather (E e) prec 33 special (id-hook SMT_Symbol (+))] .
op _*_ : Integer Integer -> Integer
[gather (E e) prec 31 special (id-hook SMT_Symbol (*))] .
op _-_ : Integer Integer -> Integer
[gather (E e) prec 33 special (id-hook SMT_Symbol (-))] .
op _div_ : Integer Integer -> Integer
[gather (E e) prec 31 special (id-hook SMT_Symbol (div))] .
op _mod_ : Integer Integer -> Integer
[gather (E e) prec 31 special (id-hook SMT_Symbol (mod))] .
op _<_ : Integer Integer -> Boolean
[prec 37 special (id-hook SMT_Symbol (<))] .
op _<=_ : Integer Integer -> Boolean
[prec 37 special (id-hook SMT_Symbol (<=))] .
op _>_ : Integer Integer -> Boolean
[prec 37 special (id-hook SMT_Symbol (>))] .
op _>=_ : Integer Integer -> Boolean
[prec 37 special (id-hook SMT_Symbol (>=))] .
op _===_ : Integer Integer -> Boolean

<!-- manual page 428 -->
416 CHAPTER 16. SMT SOLVING
[gather (e E) prec 51 special (id-hook SMT_Symbol (===))] .
op _=/==_ : Integer Integer -> Boolean
[gather (e E) prec 51 special (id-hook SMT_Symbol (=/==))] .
op _?_:_ : Boolean Integer Integer -> Integer
[gather (e e e) prec 71 special (id-hook SMT_Symbol (ite))] .
op _divisible_ : Integer Integer -> Boolean
[prec 51 special (id-hook SMT_Symbol (divisible))] .
endfm
Here a different sort Integer is given for the integer constants...,−2,−1,0,1,2,... to
avoid ambiguity with the integer constants of Maude’s Int sort. Note that _divisible_ has
theoppositeargumentordertoMaude’s_divides_operatoronIntbutitfollowstheSMT-LIB
convention.
16.3 Formulas using rational linear arithmetic
The SMT-LIB Reals theory is represented by the following signature:
fmod REAL is
protecting BOOLEAN .
sort Real .
op <Reals> : -> Real [special (id-hook SMT_NumberSymbol (reals))] .
op -_ : Real -> Real [special (id-hook SMT_Symbol (-))] .
op _+_ : Real Real -> Real
[gather (E e) prec 33 special (id-hook SMT_Symbol (+))] .
op _*_ : Real Real -> Real
[gather (E e) prec 31 special (id-hook SMT_Symbol (*))] .
op _-_ : Real Real -> Real
[gather (E e) prec 33 special (id-hook SMT_Symbol (-))] .
op _/_ : Real Real -> Real
[gather (E e) prec 31 special (id-hook SMT_Symbol (/))] .
op _<_ : Real Real -> Boolean
[prec 37 special (id-hook SMT_Symbol (<))] .
op _<=_ : Real Real -> Boolean
[prec 37 special (id-hook SMT_Symbol (<=))] .
op _>_ : Real Real -> Boolean
[prec 37 special (id-hook SMT_Symbol (>))] .
op _>=_ : Real Real -> Boolean
[prec 37 special (id-hook SMT_Symbol (>=))] .
op _===_ : Real Real -> Boolean
[gather (e E) prec 51 special (id-hook SMT_Symbol (===))] .
op _=/==_ : Real Real -> Boolean
[gather (e E) prec 51 special (id-hook SMT_Symbol (=/==))] .
op _?_:_ : Boolean Real Real -> Real
[gather (e e e) prec 71 special (id-hook SMT_Symbol (ite))] .
endfm
Here a different sort Real is given for the rational constants that look like−1/1,−3/100,
0/1, 7/3,...toavoidambiguitywiththerationalconstantsofMaude’sRatsort. The/symbolis
mandatory and servers to distinguish the reals from the integers (which are completely different

<!-- manual page 429 -->
16.4. FORMULAS USING RATIONAL AND INTEGER LINEAR ARITHMETIC 417
types in SMT-LIB). Again all of the operators are inert, although currently, rational constants
are canonicalized (this may change).
16.4 Formulas using rational and integer linear arithmetic
Finally, the SMT-LIB Reals-Ints theory is represented by the following signature, combining
both theories and two conversion functions between them plus a Boolean operation.
fmod REAL-INTEGER is
protecting INTEGER .
protecting REAL .
op toReal : Integer -> Real [special (id-hook SMT_Symbol (toReal))] .
op toInteger : Real -> Integer [special (id-hook SMT_Symbol (toInteger))] .
op isInteger : Real -> Boolean [special (id-hook SMT_Symbol (isInteger))] .
endfm
16.5 Satisfiability of formulas
The SMT solver is invoked internally using the check command. The syntax of this command
is:
check [ in ⟨ModId⟩ : ] ⟨Term⟩.
where Term must only contain the constants and operators from the appropriate signatures
above, together with variables from the three sorts, Boolean, Integer and Real when these
are available. Adding any super or subsorts to these three sorts (or subsort relations between
them) will cause inconsistency issues, as will adding any additional operators.
Maude responds with whatever the SMT solver returns, typically sat for satisfiable or
unsat for unsatisfiable. Note that the current implementation is quite brittle. There is only
a cursory attempt to check that the query to the SMT solver makes sense and no attempt to
catch exceptions. Consider the following example.
fmod TEST-RI is
protecting REAL-INTEGER .
vars W X Y Z : Boolean .
vars I J K L : Integer .
vars P Q R S : Real .
endfm
Maude> check toInteger(R) + toInteger(P) === toInteger(R + P) .
Result from sat solver is: sat
Maude> check not(toInteger(R) + toInteger(P) === toInteger(R + P)) .
Result from sat solver is: sat
Maude> check -2 < I and -2 * I > -1 and I =/== -1 .
Result from sat solver is: sat
Maude> check -2 < I and -2 * I > -1 and I =/== -1 and I - I =/== I .
Result from sat solver is: unsat

<!-- manual page 430 -->
418 CHAPTER 16. SMT SOLVING

