# Chapter 10: Strategy Language (manual pages 309-330)

<!-- manual page 309 -->
Chapter 10
Strategy Language
Rulerewritingisahighlynondeterministicprocess: ateverystep,manyrulescouldbeappliedat
various positions. A finer control on rule application is sometimes desirable, and even required,
when rules are not terminating or not convergent. Maude programmers already had resources
to restrain rewriting, from adding more information to the data representation to applying
rules explicitly at the metalevel, as described in Section 17.7. However, these methods make
specifications harder to understand and programming at the metalevel is a cumbersome task
for novice users. For these reasons, a strategy language has been proposed [86, 52, 87] as a
specification layer above those of equations and rules. This provides a cleaner way to control the
rewriting process respecting the separation of concerns principle. That is, the rewrite theory is
not modified in any way: strategies provide an additional specification level above that of rules,
so that the same system module may be executed according to different strategy specifications.
The design of Maude’s strategy language has been influenced, among others, by ELAN [12] and
Stratego [15].
Ofcourse, themostbasicactionofthestrategylanguageisruleapplication, whichisinvoked
by mentioning the rule label. More complex strategies, involving several, or unboundedly many,
rule applications, can be built by means of various strategy combinators. The language is
described in detail in Section 10.1. Moreover, strategy expressions can be given a name to be
later invoked with arguments, even recursively. Strategy modules, the modular way in which
strategies are declared, are explained in Section 10.2. Like functional and system modules,
strategy modules can be parameterized, with specific features that are described in Section 10.3.
A discussion about the strategy search commands follows in Section 10.4. Some other aspects
are covered in specific chapters, like the metarepresentation of strategies in Section 17.3 and
how strategy executions can be traced and debugged in Section 21.
Let us start with a simple example. The HANOI module below specifies the Tower of Hanoi
puzzle, invented by the French mathematician Éduard Lucas in 1883 [81]. His story tells about
an Asian temple where there are three diamond posts. The first one is surrounded by sixty-four
golden disks of increasing size from the top to the bottom. The monks are committed to move
them from one post to another respecting two rules: only a disk can be moved at a time, and
they can only be laid either on a bigger disk or on the floor. Their objective is to move all of
them to the third post, and then the world will end.
mod HANOI is
protecting NAT-LIST .
sorts Post Hanoi Game .
subsort Post < Hanoi .
297

<!-- manual page 310 -->
298 CHAPTER 10. STRATEGY LANGUAGE
op (_) [_] : Nat NatList -> Post [ctor] .
op empty op __ : -> Hanoi [ctor] .
: Hanoi Hanoi -> Hanoi [ctor assoc comm id: empty] .
vars S T D1 D2 N : Nat .
vars L1 L2 : NatList .
vars H H’ : Hanoi .
crl [move] : (S) [L1 D1] (T) [L2 D2] => (S) [L1] (T) [L2 D2 D1] if D2 > D1 .
[move] : (S) [L1 D1] (T) [nil] => (S) [L1] (T) [D1] .
rl endm
Here, the golden disks are modeled as natural numbers describing their size, and the posts are
represented as lists of disks in bottom up order. We try to rewrite the initial puzzle setting
rew (0)[3 2 1] (1)[nil] (2)[nil] .
but the command does not terminate: the disks are being moved in a loop. To prevent such a
situation, strategies can be useful. The Maude command for rewriting with strategies is:
srewrite [n] in ⟨ModId⟩ : ⟨Term⟩ by ⟨StrategyExpression⟩.
It rewrites the term according to the given strategy expression and prints all the results.
Since strategies need not be deterministic, many results may be obtained. Like in the standard
rewriting commands, we can optionally specify the module where to rewrite after in, and a
bound n on the number of solutions to be shown just after the command keyword, which can
be shortened to srew. For example,
Maude> srew [3] in HANOI : (0)[3 2 1] (1)[nil] (2)[nil] using move .
Solution 1
rewrites: 1 in 0ms cpu (0ms real) (~ rewrites/second)
result Hanoi: (0)[3 2] (1)[1] (2)[nil]
Solution 2
rewrites: 2 in 0ms cpu (1ms real) (~ rewrites/second)
result Hanoi: (0)[3 2] (1)[nil] (2)[1]
No more solutions.
rewrites: 2 in 0ms cpu (1ms real) (~ rewrites/second)
The two results of applying the move rule to the initial term are shown. Every time we request
all solutions or more solutions than possible, the interpreter indicates that there are no more
solutions when the search finishes. The order in which solutions appear is implementation-
dependent. Following the output of the srewrite command, we can view a strategy expression
α as a transformation TΣ →P(TΣ) from an initial term t to a set of terms being the results
of α in t. To elaborate more complex strategies, we need to introduce the complete strategy
language.
10.1 The strategy language
As we have anticipated, rule application is the essential building block of the strategy language.
Besidestherulelabel, furtherrestrictionscanbeimposed. Itsmostgeneralsyntaxhastheform:
label[X1 <- t1, ..., Xn <- tn]{α1, ..., αm}

<!-- manual page 311 -->
10.1. THE STRATEGY LANGUAGE 299
Between square brackets, we can optionally set an initial ground substitution for the variables
X1,...,Xn appearing in the rule. And when invoking a conditional rule with rewriting con-
ditions we must provide between curly brackets the strategies α1,...,αm to control rewriting
each of them. Only rules with exactly m condition fragments will be tried, so all rewriting
fragments must be given a strategy. For example, we can select to which Hanoi tower the disk
is moved by giving the command:
Maude> srew (0)[3 2 1] (1)[nil] (2)[nil] using move[T <- 2] .
Solution 1
rewrites: 1
result Hanoi: (0)[3 2] (1)[nil] (2)[1]
No more solutions.
rewrites: 1
Constrained by the substitution, the rewriting engine tries to match with extension (see Sec-
tion 4.8) against the lefthand sides (S)[L1 D1] (2)[L2 D2] for the first move rule, and (S)[L1
D1] (2)[nil] for the second move rule, but only the second succeeds and the term is rewritten
to solution 1.
An additional combinator all triggers a single rewriting step with all rules available in
the module, even those which are not given a label, but excluding those marked with nonexec,
external objects and implicit rules. Rewriting condition fragments are executed like in the usual
rewriting engine, without any restriction. To illustrate all and how strategies can control rule
conditions, the following module HANOI-COUNT introduces a pair operator of Game for keeping
track of the number of moves used to reach a solution.
mod HANOI-COUNT is
protecting HANOI .
op <_,_> : Hanoi Nat -> Game [ctor] .
vars H H’ : Hanoi .
var N : Nat .
crl [step] : < H, N > => < H’, s N > if H => H’ .
rl [cancel] : N => 0 [nonexec] .
rl [inc] : N => s N [nonexec] .
endm
By controlling the H => H’ condition of step with all, we indicate that only one rule appli-
cation has to be performed in each game step, so that the count is calculated correctly. The
other rules cancel and inc will be used later to count moves in a different way.
Maude> srew < (0)[3 2 1] (1)[nil] (2)[nil], 0 > using step{all} .
Solution 1
result Game: < (0)[3 2] (1)[1] (2)[nil],1 >
Solution 2
result Game: < (0)[3 2] (1)[nil] (2)[1],1 >
No more solutions

<!-- manual page 312 -->
300 CHAPTER 10. STRATEGY LANGUAGE
By default, rule applications are tried anywhere within the subject term. However, they
can be limited to the topmost position using the top restriction. For example, if we apply the
cancel rule, which rewrites any natural number1 to zero, we obtain multiple solutions:
Maude> srew 1 using cancel .
Solution 1
result Zero: 0
Solution 2
result NzNat: 1
No more solutions.
In this case, the rule can be applied both on 1= s 0, producing 0, and on its subterm 0,
producing s 0. Instead, using top the rule is only applied at the top position s 0:
Maude> srew 1 using top(cancel) .
Solution 1
result Zero: 0
No more solutions.
Combining top with the matchrew operator described in Section 10.1.2, rules can be applied
to any specified subterm in the subject term.
The other basic component of the language are the tests. They can be used for testing a
condition on the subject term. Their syntax has the form:
match P s.t. C
where P is a pattern and C is an equational condition, as described in Section 4.3. Their
syntax and behavior are similar to those of the match commands in Section 4.9: the such-that
clause can be omitted if no condition is imposed, and the starting keyword indicates where
the matching is done: on top (match), on any fragment of the flattened top modulo axioms
(xmatch), or anywhere within the subject term (amatch). The condition may refer to variables
in the pattern, whose value will be obtained from the matching. On a successful match and
condition check, the result is the initial term. Otherwise, the test does not provide any solution.
For example, we can check whether the towers of Hanoi puzzle is solved with tests:
Maude> srew (0)[nil] (1)[nil] (2)[3 2 1] using match (N)[3 2 1] H s.t. N =/= 0 .
Solution 1
result Hanoi: (0)[nil] (1)[nil] (2)[3 2 1]
No more solutions.
Maude> srew (0)[nil] (1)[nil] (2)[3 2 1] using xmatch (0)[nil] (2)[3 2 1] .
Solution 1
result Hanoi: (0)[nil] (1)[nil] (2)[3 2 1]
No more solutions.
1The decimal representation of natural numbers is just syntactic sugar for their Peano notation, whose
constructors are 0 and the successor s, as explained in Section 8.3. Hence, a natural number is a tower of s
symbols followed by a 0, which are efficiently supported using the iter attribute (see Section 4.4.2).

<!-- manual page 313 -->
10.1. THE STRATEGY LANGUAGE 301
Strategy ζ Results ζ (θ,t)
idle {t}
fail ∅
rlabel[ρ] {t′∈TΣ |t→ρ(l)→ρ(r) t′for any l→rlabel r∈R}
α;β t′∈α (θ,t) β (θ,t′)
α|β α (θ,t) ∪ β (θ,t)
α* ∞
n=0 α n(θ,t)
match P s.t. C {t} if matches(P,t,C,θ) ̸= ∅
∅ otherwise
α? β : γ
α;β (θ,t) if α (θ,t) ̸= ∅
γ (θ,t) if α (θ,t) = ∅
matchrew P s.t. C by
X1 using α1, ...,
Xn using αn
σ∈matches(P,t,C,θ) t1∈α1 (σ,σ(X1))···
tn∈αn (σ,σ(Xn)) σ[x1/t1,...,xn/tn](P)
slabel(t1,...,tn) (lhs,δ,C)∈Defs σ∈matches(slabel(t1,...,tn),lhs,C,id) δ (σ,t)
Table 10.1: Main strategy combinators and their informal semantics
Since the xmatch test matches inside the top symbol with extension, and the two posts in the
pattern form a fragment of the associative and commutative symbol on the subject term, the
second command succeeds. The same command match will fail the match, because the pattern
is not the whole term.
Maude> srew (0)[nil] (1)[nil] (2)[3 2 1] using match (0)[nil] (2)[3 2 1] .
No solution.
With the amatch variant, we can check whether the disks of radius 3 and 1 are the same post
with the following command:
Maude> srew in HANOI : (0)[nil] (1)[nil] (2)[3 2 1] using amatch 3 L1 1 .
Solution 1
result Hanoi: (0)[nil] (1)[nil] (2)[3 2 1]
No more solutions.
Once we have rule application and tests, the basic actions and control of the language, we can
combine them to build more complex strategies. Table 10.1 summarizes all primitive strategy
constructors and informally describes their semantics2
.
10.1.1 Basic control combinators
To control the flow of rule execution some combinators are defined. Let α, β and γ represent
arbitrary strategy expressions.
2We denote by ζ (θ,t) the results of evaluating the strategy expression ζin the term twith a variable context
substitution θ, by matches(P,t,C,θ) the matches of the term t into the pattern P satisfying the condition C
and extending the bindings of the previous substitution θ, and by Defs the set of strategy definitions. In the
remainder of this section, we will explain in detail all these combinators.

<!-- manual page 314 -->
302 CHAPTER 10. STRATEGY LANGUAGE
1. The concatenation α;β executes the strategy αand then the strategy β on each αresult.
In our example, we can move twice using the strategy move ; move.
Maude> srew (0)[3 2 1] (1)[nil] (2)[nil] using move ; move .
Solution 1
result Hanoi: (0)[3 2 1] (1)[nil] (2)[nil]
Solution 2
result Hanoi: (0)[3] (1)[1] (2)[2]
Solution 3
result Hanoi: (0)[3 2] (1)[nil] (2)[1]
Solution 4
result Hanoi: (0)[3] (1)[2] (2)[1]
Solution 5
result Hanoi: (0)[3 2] (1)[1] (2)[nil]
No more solutions.
Notice that the initial state appears first in the solutions. This suggest why the initial
rewrite command did not terminate.
2. The disjunction or alternative α|β executes α or β. In other words, the results of α|β
are both those of α and those of β.
Maude> srew (0)[3 2] (1)[1] (2)[nil] using move[S <- 0] | move[T <- 0] .
Solution 1
result Hanoi: (0)[3 2 1] (1)[nil] (2)[nil]
Solution 2
result Hanoi: (0)[3] (1)[1] (2)[2]
No more solutions.
3. The iteration α* runs α zero or more times consecutively. If we issue the command
Maude> srew (0)[3 2 1] (1)[nil] (2)[nil] using move * .
Solution 1
result Hanoi: (0)[3 2 1] (1)[nil] (2)[nil]
...
Solution 27
result Hanoi: (0)[nil] (1)[1] (2)[3 2]
No more solutions.
we obtain the 27 admissible states, where the disks in each post are in the correct order.
4. The concatenation, alternative, and iteration combinators resemble the similar construc-
tors for regular expressions. The empty word and empty language constants are here
represented by the idle and fail operators. The result of applying idle is always the
initial term, while fail generates no solution.

<!-- manual page 315 -->
10.1. THE STRATEGY LANGUAGE 303
Maude> srew (0)[3 2 1] using idle .
Solution 1
result Post: (0)[3 2 1]
No more solutions.
Maude> srew (0)[3 2 1] using fail .
No solution.
In a wider sense, we say that a strategy fails when no solution is obtained. Failures can
happen in less explicit situations like vacuous rule applications, unsatisfied tests, etc.
5. A conditional strategy is written α? β : γ. It executes α and then β on its results, but
if α does not produce any, it executes γ on the initial term. That is, α is the condition;
β the positive branch, which applies to the results of α; and γ the negative branch, which
is applied only if α fails.
For example, we can instruct the monks to move disks to the first and second tower only
if no disk can be moved to the target post. In this way, we obtain only 21 results out of
27.
Maude> srew in HANOI : (0)[3 2] (1)[1] (2)[nil] using
(move[T <- 2] ? idle : move) * .
Solution 1
result Hanoi: (0)[3 2] (1)[1] (2)[nil]
...
Solution 21
result Hanoi: (0)[nil] (1)[1] (2)[3 2]
No more solutions.
In the example above, we are giving precedence to one strategy over another, so that the
second is only executed if the first fails. This is a common control pattern, and so it is defined
as a derived operator with its own name, along with some other usual constructions:
• The or-else combinator is defined by
αor-else β ≡α? idle : β
As shown in the example above, it executes β only if α has failed.
• The non-void iteration α+ is defined as α+ ≡α;α*.
• The negation is defined as:
not(α) ≡α? fail : idle
It fails when α succeeds, and succeeds as an idle when α fails.
• The normalization operator
α! ≡α* ; not(α)
applies α until it cannot be further applied.
Maude> srew (0)[3 2 1] (1)[nil] (2)[nil] using move[S <- 0] ! .
Solution 1

<!-- manual page 316 -->
304 CHAPTER 10. STRATEGY LANGUAGE
result Hanoi: (0)[3] (1)[1] (2)[2]
Solution 2
result Hanoi: (0)[3] (1)[2] (2)[1]
No more solutions.
• The try strategy is defined by try(α) ≡α? idle : idle. It applies α, but if it fails, it
returns the initial state.
• The test strategy, defined by test(α) ≡not(not(α)), fails whenever αfails, but when
α succeeds, it returns the initial term instead of its results.
Maude> srew (0)[3] (1)[1] (2)[2] using test(move[S <- 0]) .
No solution.
Maude> srew (0)[3] (1)[1] (2)[2] using test(move[S <- 1]) .
Solution 1
result Hanoi: (0)[3] (1)[1] (2)[2]
No more solutions.
As another example of the use of the concatenation operator, let us go back to the rent-
a-car example presented in Section 6.2. Since the specification has several non-executable
rules, because there are extra variables in their righthand sides, it cannot be rewritten without
appropriate strategies specifying an assignment for these variables.
Given the following RENT-A-CAR-STORE-TEST module, in which an initial configuration is
provided, you can find below a rewrite using a strategy that guides the execution. Specifically,
it begins by making the user ’C1 rent the mid size car ’A3 for two days, and then renting the
full size car ’A5 one day after the previous one is returned, for one day; since the car is returned
late, the client is suspended, and two days later pays a fine.
mod RENT-A-CAR-STORE-TEST is
pr RENT-A-CAR-STORE .
op StoreConf : -> Configuration [memo] .
eq StoreConf
= < ’C1 : Customer | cash : 5000, debt : 0, suspended : false >
< ’C2 : Customer | cash : 5000, debt : 0, suspended : false >
< ’A1 : EconomyCar | available : true, rate : 100 >
< ’A3 : MidSizeCar | available : true, rate : 150 >
< ’A5 : FullSizeCar | available : true, rate : 200 >
< ’C : Calendar | date : 0 > .
endm
Maude> srew StoreConf
using car-rental[U:Oid <- ’C1, I:Oid <- ’A3, NumDays:Int <- 2,
A:Oid <- ’a0] ;
---- client C1 rents a mid
---- size car A3 for 2 days
new-day ;
new-day ;
on-date-car-return ; new-day ;
---- car A3 is returned

<!-- manual page 317 -->
10.1. THE STRATEGY LANGUAGE 305
car-rental[U:Oid <- ’C1, I:Oid <- ’A5, NumDays:Int <- 1,
A:Oid <- ’a1] ;
---- client C1 rents the full
---- size car A5 for 1 day
new-day ; new-day ;
late-car-return ; new-day ;
suspend-late-payers ; new-day ;
new-day ;
---- two days pass
---- car A5 is returned
---- client C1 is suspended
pay-debt[Amnt:Int <- 100] . ---- client C1 pays 100$
Solution 1
rewrites: 54 in 0ms cpu (0ms real) (178217 rewrites/second)
result Configuration:
< ’A1 : EconomyCar | available : true, rate : 100 >
< ’A3 : MidSizeCar | available : true, rate : 150 >
< ’A5 : FullSizeCar | available : true, rate : 200 >
< ’C : Calendar | date : 8 >
< ’C1 : Customer | cash : 4400, debt : 140, suspended : true >
< ’C2 : Customer | cash : 5000, debt : 0, suspended : false >
No more solutions.
10.1.2 Rewriting of subterms
The match and rewrite operator matchrew restricts the application of a strategy to a specific
subterm of the subject term. Its syntax is
matchrew P(X1,...,Xn) s.t. C by X1 using α1, ..., Xn using αn
where P is a pattern with variables X1,...,Xn among others, and C is an optional equational
condition. The usingclauses associate variables in the pattern, which are matched by subterms
of the matched term, with strategies that will be used to rewrite them. These variables must
be distinct and must appear in the pattern. As in the case of tests, there are three flavors for
this operator, depending on the type of matching: matchrew, xmatchrew, and amatchrew.
The semantics of this operator is illustrated in Figure 10.1. All matches of the pattern in
the subject term are tried, checking the condition if any. If none succeeds, the strategy fails.
Otherwise, for each match of the main pattern the subterms matching each Xi are extracted,
rewritten according to αi, and their solutions are put in place of the original subterm. The
rewriting of the subterms happens in parallel, in the sense that the terms evolve in a completely
independent and fair way. Finally, all possible combinations of the subterm results will give
rise to different reassembled solutions.
Another advantage of matchrew is that it let us obtain information about the term and
use it to modify the behavior of the strategies. The scope of the variables in the pattern
and the condition is extended to the substrategies α1,...,αn, so that the values they take by
matching and evaluation can be used in nested rule substitutions, strategy calls, and match and
matchrew operators. Conversely, when these variables are bound by outer constructs, they keep
their values and matching is done against a partially instantiated pattern. When a variable in
the condition is not bound either by the outer context or by the matching, the strategy will be
unusable and a warning will be shown by the interpreter.

<!-- manual page 318 -->
306 CHAPTER 10. STRATEGY LANGUAGE
f(... g(...) ...) → f(... g(...) ...)
matching substitution
g(...t...s...) → g(...t′
...s′...)
αi αj
rewriting
Figure 10.1: Behavior of the amatchrew combinator
For example, instead of using the step rule to count the moves in the Tower of Hanoi, we
could have used a matchrew and the increment rule inc. This is written:
Maude> srew [1] in HANOI-COUNT : < (0)[3 2 1] (1) [nil] (2)[nil], 0 >
using (matchrew < H, N > by H using move, N using top(inc)) * ;
amatch (2)[3 2 1] .
Solution 1
rewrites: 523 in 56ms cpu (56ms real) (9339 rewrites/second)
result Game: < (0)[nil] (1)[nil] (2)[3 2 1], 7 >
Notice the use of top in the inc rule application. As we have seen in the previous example
with the cancel rule in page 298, inc could have been applied inside successors if we had
omitted the top modifier. Although in this particular case the result would not have been
affected, unnecessary work would have been done, as we can observe in the rewrites count of
the following execution.
Maude> srew 1 using inc .
Solution 1
rewrites: 1 in 0ms cpu (0ms real) (~ rewrites/second)
result NzNat: 2
No more solutions.
rewrites: 2 in 0ms cpu (0ms real) (~ rewrites/second)
10.1.3 The one operator
Strategies are nondeterministic due to the different possible matches of the rules and the sub-
term rewriting operator, disjunction, iteration, etc. The various rewriting paths that these
alternatives make possible are explored almost simultaneously by the strategic search engine.
Sometimes, the different options lead to the same result or, the results being different, it may
not matter which of them is selected. For efficiency purposes, the one combinator is introduced
to execute its argument strategy and return only one of its solutions, namely the first one found.
If the strategy does not produce any solutions, neither does one.
Maude> srew (0)[3 2 1] (1)[nil] (2)[nil] using one(move * ; amatch (2)[3 2 1]) .
Solution 1
rewrites: 121
result Hanoi: (0)[nil] (1)[nil] (2)[3 2 1]
No more solutions.
rewrites: 121

<!-- manual page 319 -->
10.2. STRATEGY MODULES 307
Maude> srew (0)[3 2 1] (1)[nil] (2)[nil] using move * ; amatch (2)[3 2 1] .
Solution 1
rewrites: 121
result Hanoi: (0)[nil] (1)[nil] (2)[3 2 1]
No more solutions.
rewrites: 150
The most noticeable performance improvements are obtained the closer one is applied to rules,
because the alternatives are pruned immediately. However, this should be done only when
paths can be discarded safely.
Maude> srew (0)[3 2 1] (1)[nil] (2)[nil] using one(move) * .
Solution 1
result Hanoi: (0)[3 2 1] (1)[nil] (2)[nil]
Solution 2
result Hanoi: (0)[3 2] (1)[1] (2)[nil]
No more solutions.
In the execution above, one(move) tries only a single move at each iteration, and the resulting
sequence of moves does not lead to the solved game (in fact, it loops after the first move).
10.1.4 Strategy calls
Strategies with their own name and taking parameters can be defined in strategy modules, which
will be explained in Section 10.2. These named strategies can be called as a strategy expression
with a typical prefix operator syntax: a strategy identifier followed by a comma-separated list of
arguments between parentheses. When the strategy does not take any arguments and there is
no rule with the same name, the parentheses can be omitted. Suppose we already have defined
a strategy moveAll to move disks between certain posts indicated by its arguments, with its
third argument being the number of disks to be moved. It can be applied to an initial state
like this:
Maude> srew (0)[3 2 1] (1)[nil] (2)[nil] using moveAll(0, 2, 3) .
Solution 1
rewrites: 24 in 4ms cpu (2ms real) (6000 rewrites/second)
result Hanoi: (0)[nil] (1)[nil] (2)[3 2 1]
No more solutions.
rewrites: 24 in 4ms cpu (2ms real) (6000 rewrites/second)
How this and other strategies are defined is explained in the next section.
10.2 Strategy modules
Strategy modules let the user give name to strategy expressions. Named strategies facilitate the
use and description of complex strategies, and they extend the expressiveness of the language

<!-- manual page 320 -->
308 CHAPTER 10. STRATEGY LANGUAGE
by means of recursive and mutually recursive strategies. As for functional and system modules,
a strategy module is declared in Maude using the keywords
smod ⟨ModuleName⟩ is ⟨DeclarationsAndStatements⟩ endsm
A typical strategy module would import the system module it will control, declare some strate-
gies, and define them by means of strategy definitions. All statements and declarations that
were available in functional and system modules can also be included in strategy modules, but
we encourage to avoid them as much as possible to emphasize a clean separation between the
rewriting theory and the strategies that control it.
For example, the following strategy module HANOI-SOLVE defines the recursive strategy
moveAll that we have used in Section 10.1.4 to illustrate strategy calls and solve the puzzle.
smod HANOI-SOLVE is
protecting HANOI .
protecting HANOI-AUX .
strat moveAll : Nat Nat Nat @ Hanoi .
vars S T C M : Nat .
sd moveAll(S, S, C) := idle .
sd moveAll(S, T, 0) := idle .
sd moveAll(S, T, s(C)) := moveAll(S, third(S, T), C) ;
move[S <- S, T <- T] ;
moveAll(third(S, T), T, C) .
endsm
The operator third, imported from the following functional module HANOI-AUX, calculates the
third with respect to two given posts. In general, when we require some auxiliary operations
to specify a strategy, we can define them in an auxiliary module that is then imported by the
strategy module.
fmod HANOI-AUX is
protecting SET{Nat} .
op third : Nat Nat ~> Nat .
vars N M K : Nat .
ceq third(N, M) = K if N, M, K := 0, 1, 2 .
endfm
The features making up a strategy module are strategy declarations and definitions. Strategies
are declared as follows:
strat ⟨StratName⟩+ : ⟨Sort-1⟩... ⟨Sort-k⟩ @ ⟨Sort⟩ [⟨StratAttributes⟩] .
Strategy names are single word identifiers. The sorts between the colon and the @ sign are
the argument sorts of the strategy. The last argument after the @ sign is the subject sort, i.e.,
the sort of the terms to which the strategy will be applied. The colon can be omitted when a
strategy has no arguments. Multiple strategy names, separated by blank space, can be used to
define multiple strategies at once, provided that all have the same argument sorts and subject
sort. In this case the plural keyword strats instead of strat is recommended. The only
admitted strategy attribute is metadata (see Section 4.5.2).

<!-- manual page 321 -->
10.2. STRATEGY MODULES 309
Strategy names can be overloaded, i.e., strategies can be defined with the same name but
may have different arities and argument types. Like operators, strategies are defined at the
kind level. Homonymous strategies with the same arity and such that for each argument the
corresponding sorts have the same kind are considered the same. In practice, the concrete
sorts of the strategy signature do not have any effect and should be considered as a declaration
of intent. Repeated declarations of the exact same strategy twice (with the same name and
argument kinds) will generate a warning. For example, when processing the module
smod STRATS is
protecting INT .
strat repeat : Nat @ Nat .
strat repeat : Int @ Nat .
endsm
the interpreter will issue the warning:
Warning: <standard input>, line 5 (smod STRATS): strategy declaration
strat repeat : Int @ Nat . conflicts with
strat repeat : Nat @ Nat . from <standard input>, line 4 (smod STRATS).
For its part, the subject sort is ignored to all effects: it is not checked when calling the strategy
and not used to distinguish strategies.
Without a corresponding definition, named strategies behave like fail. Definition state-
ments should be provided to properly define each named strategy according to the syntax:
sd ⟨StrategyCall⟩ := ⟨StrategyExpression⟩ [⟨SdAttributes⟩] .
The lefthand side of a definition is a strategy call, as seen in the previous section, probably
containing variables, which can be used in the righthand side. In fact, all variables in the
strategy expression, except variables in matching patterns, must appear in the lefthand side.
Like equations and rules, strategy definitions may be conditional:
csd ⟨StrategyCall⟩ := ⟨StrategyExpression⟩ if ⟨Condition⟩ [⟨SdAttributes⟩] .
The condition of a strategy definition is an equational one, as described in Section 4.3, i.e.,
rewriting conditions t1 => t2 are not allowed. The attributes metadata, label and print
(see Section 4.5) can be attached to sd and csd definitions. When a strategy is invoked, all
availabledefinitionsforthenamearetried, andallthosewhoselefthandsidematchesthecalling
arguments are executed. Moreover, conditions may generate different variable assignments, in
which case the strategy will be applied with each one of them. For example, we could have
given the third definition of moveAll in HANOI-SOLVE above as
csd moveAll(S, T, s(C)) := moveAll(S, third(S, T), C) ;
move[S <- S, T <- T] ;
moveAll(third(S, T), T, C) if S =/= T .
since the definition does not make sense when the target and source towers are the same. In
fact, this case always fails because the partial function third(S, T) evaluates to an error term
of sort [Nat] such that the first recursive call does not match any definition.
To conclude this section, let us show another example: the insertion-sort algorithm imple-
mented by means of strategies. The underlying array is represented as a set of pairs containing
both a position and a value. It is assumed that there is a single entry for each position between
one and the length of the array. A rule swap is defined to swap the values between two entries.

<!-- manual page 322 -->
310 CHAPTER 10. STRATEGY LANGUAGE
mod SWAPPING{X :: DEFAULT} is
protecting ARRAY{Nat, X} .
vars I J : Nat .
vars V W : X$Elt .
var AR : Array{Nat, X} .
rl [swap] : J |-> V ; I |-> W => J |-> W ; I |-> V .
op maxIndex : Array{Nat, X} ~> Nat .
eq maxIndex(empty) = 0 .
eq maxIndex(I |-> V ; AR) = if maxIndex(AR) < I then I else maxIndex(AR) fi .
endm
Then, the insertion-sort algorithm is specified as the insort strategy. It visits each position
of the array in increasing order, maintaining the invariant that the entries to the left of the
current index are sorted. At each iteration, it inserts the current value in the sorted fragment
of the array. Insertion is done by the insert strategy, which compares in descending order each
entry with its neighbor and swaps them if they are misplaced. The base case is the insertion
in the first position, which does not require any work.
view DEFAULT+ from DEFAULT to STRICT-TOTAL-ORDER + DEFAULT is
endv
smod INSERTION-SORT{X :: STRICT-TOTAL-ORDER + DEFAULT} is
protecting SWAPPING{DEFAULT+}{X} * (
sort Array{Nat, DEFAULT+}{X} to NatArray{X}
) .
strat swap : Nat Nat @ NatArray{X} .
strats insert insort : Nat @ NatArray{X} .
vars X Y J I : Nat .
vars V W : X$Elt .
var AR : NatArray{X} .
sd insort(Y) := try(match AR s.t. Y <= maxIndex(AR) ;
insert(Y) ;
insort(Y + 1)) .
sd insert(1) := idle [label base-case] .
csd insert(s(X)) := try(xmatch X |-> V ; s(X) |-> W s.t. W < V ;
swap(X, s(X)) ;
insert(X))
if X > 0 [label recursive-case] .
sd swap(X, Y) := swap[J <- X, I <- Y] .
endsm
view Int<0 from STRICT-TOTAL-ORDER + DEFAULT to INT is
sort Elt to Int .
endv
smod INSERTION-SORT-INT is
protecting INSERTION-SORT{Int<0} .
endsm

<!-- manual page 323 -->
10.2. STRATEGY MODULES 311
Then, we can sort any array using insort with the following srewrite command:
Maude> srewrite 1 |-> 8 ; 2 |-> 3 ; 3 |-> 15 ; 4 |-> 5 ; 5 |-> 2 using insort(2) .
Solution 1
rewrites: 116 in 0ms cpu (0ms real) (~ rewrites/second)
result NatArray{Int<0}: 1 |-> 2 ; 2 |-> 3 ; 3 |-> 5 ; 4 |-> 8 ; 5 |-> 15
No more solutions.
rewrites: 116 in 0ms cpu (1ms real) (~ rewrites/second)
10.2.1 Module importation
Strategy modules can be seen as an additional layer of specification over functional and system
modules, which are not allowed to import strategy modules. Instead, strategy modules can im-
port functional, system, or strategy modules using the protecting, extending, and including
modes.
The semantic requirements for each inclusion mode are described in Section 7.1 for func-
tional and system modules. Similar requirements apply to strategies when importing strategy
modules. A protecting extension M of a strategy module M′ should not alter the rewriting
paths permitted by each strategy declared in M′. The declaration and definition of new strate-
gies in the importing module does not affect the semantics of the strategies declared in the
imported module, but adding new definitions for the imported strategies may change their be-
havior. If new rewriting paths are added to the imported strategies, but those of the imported
module are preserved, the extending mode should be used. In any other case, the including
mode is the right choice.
Let us illustrate the importation modes with a simple example. The following strategy
module SMOD-IMPORT-EXAMPLE declares two strategies st1 and st2. Since st1 is not given any
definition, no rewriting is possible using it, and thus st1 is equivalent to fail. On the other
hand, the definition of st2 describes all rewriting paths of the form q →’z for every quoted
identifier q.
mod SMOD-IMPORT-EXAMPLE0 is
protecting QID .
rl [ab] : ’a => ’b .
rl [bc] : ’b => ’c .
rl [zz] : Q:Qid => ’z .
endm
smod SMOD-IMPORT-EXAMPLE is
protecting SMOD-IMPORT-EXAMPLE0 .
strats st1 st2 @ Qid .
sd st2 := st1 ? bc : zz .
endsm
The module SMOD-IMPORT-EXAMPLE’ below includes a new definition for st2 that allows the
additional rewriting path ’a →’b. Hence, SMOD-IMPORT-EXAMPLE should not be imported in
protecting mode but in extending mode.
smod SMOD-IMPORT-EXAMPLE’ is
extending SMOD-IMPORT-EXAMPLE .
sd st2 := ab .
endsm

<!-- manual page 324 -->
312 CHAPTER 10. STRATEGY LANGUAGE
Although there is no direct way of restricting an already defined strategy, this may indirectly
happen due to the conditional and its derived operators. The following module extends st1
with the ’a →’b path. As a result, the condition of st1’s definition st1 ? bc : zz has now
solutions for ’a, the rewriting path ’a →’b →’c is allowed, but also ’a →’z is lost, because
the negative branch of the conditional is not evaluated.
smod SMOD-IMPORT-EXAMPLE’’ is
including SMOD-IMPORT-EXAMPLE .
sd st1 := ab .
endsm
The semantic requirements for strategies refer to the rewriting paths that can be followed while
executing the strategy, which is stronger than refering only to their evaluation results. However,
it this example, the results are also changed non-conservatively:
Maude> srew in SMOD-IMPORT-EXAMPLE : ’a using st2 .
Solution 1
rewrites: 1
result Qid: ’z
No more solutions.
rewrites: 1
Maude> srew in SMOD-IMPORT-EXAMPLE’’ : ’a using st2 .
Solution 1
rewrites: 2
result Qid: ’c
No more solutions.
rewrites: 2
Asforfunctionalandsystemmodules, theserequirementsarenotcheckedbyMaudeatruntime.
Finally, the language of module renamings (see Section 7.3) is extended with strategy re-
namings. Like for regular operators, strat name to newName renames all strategies whose
name is name to newName. To rename only a specific instance of an overloaded strategy name,
its arity and subject sort can be made explicit with strat name : s1 ...sn @ s to newName.
10.3 Parameterization in strategy modules
Strategy modules can be parameterized like any other module in Maude, by means of theories
and views, as explained in Section 7.4. A parameterized strategy module is a usual strategy
module that takes a set of formal parameters, bound to some theories:
smod SM{X1 :: T1, ..., XN :: TN} is ... endsm
These theories can be functional or system theories, or also strategy theories. A strategy theory
specifies the signature of the strategies that a concrete strategy module parameter must provide
andtherequirementsthattheseactualstrategiesmustsatisfy. Howaparticularstrategymodule
meets the requirements of a strategy theory is specified by means of a view with strategy
bindings, which will be described soon, and then views are used to instantiate parameterized
modules with actual parameters. Strategy theories are introduced with the syntax:
sth T is ... endsth

<!-- manual page 325 -->
10.3. PARAMETERIZATION IN STRATEGY MODULES 313
and have features entirely similar to those of strategy modules. They can:
• import other theories in including mode,
• import modules of any kind,
• declare strategies,
• provide strategy definitions, and
• include other declarations and statements that functional and system theories admit.
The simplest strategy theory STRIV declares a single strategy without parameters:
sth STRIV is
including TRIV .
strat st @ Elt .
endsth
The subject sort Elt of the strategy st is imported from the functional theory TRIV. Strategy
theories can never be used to parameterize functional or system modules.
Given a strategy module that meets the requirements of a strategy theory, we should specify
how they are actually met by using a view. The repertoire of mappings that can appear in a
view (see Section 7.4.2) has been extended with additional syntax for strategy bindings:
1. Mapping all formal strategies with the same given name at once. Each affected strategy
will be seeked in the target module after translating its arguments according to the sort
mappings of the view.
strat ⟨StratName⟩ to ⟨StratName⟩.
For example, consider the following strategy theory FOO and strategy module BAR:
sth STHEORY is
including TRIV .
protecting NAT .
strat foo : Elt @ Elt .
strat foo : Elt Nat @ Elt .
endsth
smod SMODULE is
protecting STRING .
protecting NAT .
strat bar : String @ String .
strat bar : String Nat @ String .
endsm
The following view Bar maps both the strategy foo(Elt) to bar(String) and the strat-
egy foo(Elt, Nat) to bar(String, Nat) with a single mapping strat foo to bar.
view SModule from STHEORY to SMODULE is
sort Elt to String .
strat foo to bar .
endv
2. Mapping a single formal strategy to an existing strategy in the target module.
strat ⟨StratName⟩ : ⟨Sort-1⟩... ⟨Sort-k⟩ @ ⟨Sort⟩ to ⟨StratName⟩.
For example, the previous view Bar can also be written as:
view SModule’ from STHEORY to SMODULE is
sort Elt to String .
strat foo : Elt @ Elt to bar .
strat foo : Elt Nat @ Elt to bar .
endv

<!-- manual page 326 -->
314 CHAPTER 10. STRATEGY LANGUAGE
3. Mapping a single formal strategy to an actual strategy expression in the target module.
strat ⟨StrategyCall⟩ to expr ⟨StrategyExpression⟩.
Only variables are allowed as arguments in the lefthand side of the mapping, as for
the operator to term mappings (see Section 7.4.2). These variables can be used in the
strategy expression in the righthand side. When the module is instantiated, the strategy
calls matching the lefthand side will be substituted by the lefthand side expression in any
strategy definition of the module.
Finally, and as usual, modules are instantiated with views using the syntax
SM{View1, ..., ViewN}
Let us illustrate how parameterized strategy modules are instantiated by strategy views
withanexample. ThefollowingparameterizedstrategymoduleBACKTRACKINGisabacktracking
problem solver: the generic algorithm is specified as a strategy solvedepending on some formal
declarations.
smod BACKTRACKING{X :: BT-STRAT} is
strat solve @ X$State .
var S : X$State .
sd solve := (match S s.t. isSolution(S)) ?
idle : (expand ; solve) .
endsm
This module is parameterized by the backtracking problem specification, whose requirements
are expressed in the strategy theory BT-STRAT. Its main element is the expand strategy, which
is intended to generate all the successors of the problem state in which it is applied.
fth BT-ELEMS is
protecting BOOL .
sort State .
op isSolution : State -> Bool .
endfth
sth BT-STRAT is
including BT-ELEMS .
strat expand @ State .
endsth
ThestrategytheoryincludesafunctionaltheoryBT-ELEMS,wherewerequireasortStateforthe
problem states, and a predicate isSolution to recognize solutions among them. The strategy
theory itself declares the strategy expand without parameters. Now, we specify a concrete
backtracking problem, the 8-queens problem, and instantiate the parameterized BACKTRACKING
module with it by means of a view. Then, the instantiated strategy solve will be able to solve
the 8-queens problem. The problem is described in the system module QUEENS and the strategy
module QUEENS-STRAT:
mod QUEENS is
protecting LIST{Nat} .
protecting SET{Nat} .
op isOk : List{Nat} -> Bool .
op ok : List{Nat} Nat Nat -> Bool .
op isSolution : List{Nat} -> Bool .

<!-- manual page 327 -->
10.3. PARAMETERIZATION IN STRATEGY MODULES 315
vars N M Diff : Nat .
var L : List{Nat} .
var S : Set{Nat} .
eq isOk(L N) = ok(L, N, 1) .
eq ok(nil, M, Diff) = true .
ceq ok(L N, M, Diff) = ok(L, M, Diff + 1)
if N =/= M /\ N =/= M + Diff /\ M =/= N + Diff .
eq isSolution(L) = size(L) == 8 .
crl [next] : L => L N if N, S := 1, 2, 3, 4, 5, 6, 7, 8 .
endm
smod QUEENS-STRAT is
protecting QUEENS .
strat expand @ List{Nat} .
var L : List{Nat} .
sd expand := top(next) ; match L s.t. isOk(L) .
endsm
Then, we define a view from the BT-STRAT strategy theory to QUEENS-STRAT:
view QueensBT from BT-STRAT to QUEENS-STRAT is
sort State to List{Nat} .
strat expand to expand .
endv
As with sorts and operators, unmentioned strategy names get the identity mapping, so the
mapping for expandin QueenBTcould be omitted. Finally, we can instantiate the parameterized
module:
smod BT-QUEENS is
protecting BACKTRACKING{QueensBT} .
endsm
and run solve for 8-queens:
Maude> srew [2] nil using solve .
srewrite [2] in BT-QUEENS : nil using solve .
Solution 1
rewrites: 285984 in 238ms cpu (238ms real) (1200211 rewrites/second)
result NeList{Nat}: 1 5 8 6 3 7 2 4
Solution 2
rewrites: 285984 in 238ms cpu (238ms real) (1200211 rewrites/second)
result NeList{Nat}: 1 6 8 3 7 4 2 5
Typically, views will be defined from a strategy theory to a strategy module or theory, but
other combinations are possible:
1. Views from strategy theories to non-strategy modules (or theories) are useful when the
target strategies are simple enough to be defined by strategy expressions, for which to
expr bindings must be used. A simple example is
view StrivIdle from STRIV to TRIV is
sort Elt to Elt . *** can be omitted

<!-- manual page 328 -->
316 CHAPTER 10. STRATEGY LANGUAGE
strat st to expr idle .
endv
2. Views from non-strategy theories to strategy modules (or theories) can be used to instan-
tiate parameterized funcional or system modules with strategy modules. In that case, the
instantiated module (or theory) will become a strategy module (or theory). These views
cannot contain strategy bindings, as there are no formal strategies to bind. For instance,
a view from the functional theory TRIV to the strategy theory STRIV can be defined as
view TRIV2STRIV from TRIV to STRIV is
sort Elt to Elt . *** can be omitted
endv
The instantiation LIST{TRIV2STRIV} then yields a parameterized strategy module taking
a parameter of the STRIV theory.
Moreover, we could have defined a view from the strategy theory BT-STRAT directly to the
functional module QUEENS by binding the formal strategy expand to the actual inline definition
of the expand strategy we already use in the strategy module QUEENS-STRAT. In this way, we
do not need to define or use the QUEENS-STRAT strategy module.
view QueensBT2 from BT-STRAT to QUEENS is
sort State to List{Nat} .
var L : State .
strat expand to expr top(next) ; match L s.t. isOk(L) .
endv
All the details about module instantiation in Section 7.4.4 are applicable to the strategy
level as well. However, the instantiation of strategies by expressions may be problematic when
applied on the left-hand side of strategy definitions, which must always be strategy calls. For
example, if the module BACKTRACKING above contained a strategy definition
sd expand := idle .
its instantiation with QueensBT2 would yield
sd top(next) ; match L s.t. isOk(L) := idle .
Since this is not valid, Maude will show a warning and reject the ill-formed definition.
10.4 Strategy search and the dsrewrite command
Although strategies control and restrict rewriting, they do not make the process deterministic.
A strategy may allow multiple rewriting paths, produced by alternative local decisions that
may appear during its execution. The rewrite and frewrite commands eliminate this non-
determinism by each choosing a specific fixed strategy, as explained in Sections 5.4.1 and 5.4.2.
Instead, the search command explores all the possible rule applications looking for a term
matching a given target. The srewrite command agrees with search in the exhaustive explo-
ration of the alternatives, looking, in this case, for strategy solutions. However, the srewrite
search is not conducted on the complete rewriting tree of search, but in a subtree pruned by
the effect of the strategy. How this tree is explored has implications for the command’s output
and its performance.
The srewrite command explores the rewriting graph following a fair policy which ensures
that all solutions reachable in a finite number of steps are eventually found, unless the inter-
preter runs out of memory. Without being a breadth-first search, multiple alternative paths are

<!-- manual page 329 -->
10.4. STRATEGY SEARCH AND THE DSREWRITE COMMAND 317
explored in parallel. This can be observed in the n-queens problem presented in Section 10.3,
and in the particular example we showed:
Maude> srew [2] nil using solve .
Solution 1
rewrites: 285984 in 249ms cpu (248ms real) (1147935 rewrites/second)
result NeList{Nat}: 1 5 8 6 3 7 2 4
Solution 2
rewrites: 285984 in 249ms cpu (249ms real) (1147935 rewrites/second)
result NeList{Nat}: 1 6 8 3 7 4 2 5
Notice that both solutions are obtained at the same time and with the same number of rewrites,
since these and even more rewriting paths are being explored in parallel. All of them increment
indiscriminately the common rewrites counter. In this particular example, being all the solu-
tions at the same depth in the rewriting tree, the tree has to be explored almost completely
before the first solution is found.
An alternative rewriting command dsrewrite(depth strategic rewrite) explores the strategy
rewriting graph in depth. Its syntax coincides with that of the srewrite command except for
the starting keyword, which can be abbreviated to dsrew.
dsrewrite [n] in ⟨ModId⟩ : ⟨Term⟩ using ⟨StrategyExpression⟩.
Usingthiscommandinthepreviousexample,wecanobservesomedifferences. Thefirstsolution
is obtained sooner and some rewrites before the second. In absolute terms, both solutions are
shown earlier and using fewer rewrites. The memory usage is reduced too.
Maude> dsrew [2] nil using solve .
dsrewrite [2] in BT-QUEENS : nil using solve .
Solution 1
rewrites: 15602 in 16ms cpu (17ms real) (942092 rewrites/second)
result NeList{Nat}: 1 5 8 6 3 7 2 4
Solution 2
rewrites: 20339 in 19ms cpu (21ms real) (1022728 rewrites/second)
result NeList{Nat}: 1 6 8 3 7 4 2 5
However, the disadvantage is that the depth-first exploration could get lost in an infinite branch
before finding some reachable solutions, which would then be missed. The depth-first search
order is determined by the order of the direct successors, which is summarized in the following
precedences:
• Rule applications and matchrew matches are issued as in the rewrite and match com-
mands, i.e., from outer to inner positions and from left to right. Rules are selected in
their syntactical order within the module.
• In an alternative α|β, the successors of α appear before the successors of β.
• In an iteration, leaving precedes continuing with another iteration.
• Strategy definitions are selected in the order they appear in the module. If the parameters
matchindifferentways, thesestrategyexecutionsaregeneratedasinthematchcommand.
• Rule and strategy condition fragments are processed from left to right and matches are
generated as before. Rewriting condition fragments solutions are depth-first searched
using these precedences.

<!-- manual page 330 -->
318 CHAPTER 10. STRATEGY LANGUAGE

