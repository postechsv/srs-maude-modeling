# Chapter 20: LaTeX Support (manual pages 517-530)

<!-- manual page 517 -->
Chapter 20
LATEX Support
Maude provides some support for LATEX. Basically, it can generate a log of the interaction with
the interpreter, including the listing of modules and views, and the output of most commands.
It is activated by the command-line flag -latex-log=⟨file⟩(see Appendix A.1) where ⟨file⟩is
the name of a LATEX file (with extension .tex) where the generated LATEX code will be written.
The Maude distribution comes with a LATEX style file (maude.sty) that contains the macros
and environments used by the generated LATEX code.
The intended use is that the resulting file can be processed with pdflatex to get a PDF file
for initial examination. Then, to include an excerpt of generated output in a larger document,
the relevant snippet of LATEX code can just be copied and pasted into the document. To be
able to compile such code the LATEX maude package must be added to the document preamble.
\usepackage{maude}
The maude.sty package generates an output with a mathematical look, using italics, math
symbols, and other features in different places. For a mathematical look without symbols like
->, =>, etc. replaced with mathematical symbols, the package has an ASCII option:
\usepackage[ASCII]{maude}
To facilitate the identification of the right excerpts, each command and its response corre-
sponds to a chunk of LATEX code in its own environment, and it is preceded by a comment with
the full text of the command that generated it. To enable easier location of blocks of LATEX
code to cut and paste, the comments which mark the output of commands look as follows:
%
% <command-with-arguments>
%
<latex code for command and results>
%
% End of <command>
%
We illustrate the appearance of the LATEX output in what follows on several examples. The
interested reader is invited to play with it and get familiar with the provided look.
20.1 Some basic first examples
Consider the session below. It is started with the latex-log flag to write the LATEX log on a file
with name output.tex. Then, after pasting the simple module SIMPLE-NAT (see Section 2.2),
505

<!-- manual page 518 -->
506 CHAPTER 20. LATEX SUPPORT
a term is reduced in it. In the interaction, we have intentionally made a mistake, so that we
can see what is shown in the log, and then the intended term is used. The interaction ends
with a show ops command right before quitting.
% maude -latex-log=output.tex
\||||||||||||||||||/
--- Welcome to Maude ---
/||||||||||||||||||\
Maude 3.5.1 built: Jul 16 2025 12:00:00
Copyright 1997-2025 SRI International
Wed Jul 16 20:00:00 2025
Maude> fmod SIMPLE-NAT is
> sort Nat .
> op zero : -> Nat .
> op s_ : Nat -> Nat .
> op _+_ : Nat Nat -> Nat .
> vars N M : Nat .
> eq zero + N = N .
> eq s N + M = s (N + M) .
> endfm
Maude> show module .
fmod SIMPLE-NAT is
sort Nat .
op zero : -> Nat .
op s_ : Nat -> Nat .
op _+_ : Nat Nat -> Nat .
vars N M : Nat .
eq zero + N = N .
eq s N + M = s (N + M) .
endfm
Maude> show desugared .
fmod SIMPLE-NAT is
including BOOL .
sort Nat .
op zero : -> Nat .
op s_ : Nat -> Nat [prec 15 gather (E)] .
op _+_ : Nat Nat -> Nat [prec 41 gather (E E)] .
var N : Nat .
var M : Nat .
eq zero + N = N .
eq s N + M = s (N + M) .
endfm
Maude> red s s 0 + s s 0 .
Warning: <standard input>, line 12: didn’t expect token 0:
s s 0 <---*HERE*
Warning: <standard input>, line 12: no parse for term.
Maude> red s s zero + s s zero .
reduce in SIMPLE-NAT : s s zero + s s zero .
rewrites: 3 in 0ms cpu (0ms real) (200000 rewrites/second)
result Nat: s s s s zero
Maude> show ops BOOL .
op if_then_else_fi : Bool Universal Universal -> Universal [poly (2 3 0) prec 0
gather (& & &) special (
id-hook BranchSymbol

<!-- manual page 519 -->
20.1. SOME BASIC FIRST EXAMPLES 507
term-hook 1 (true)
term-hook 2 (false))] .
op _==_ : Universal Universal -> Bool [poly (1 2) prec 51 gather (E E) special
(
id-hook EqualitySymbol
term-hook equalTerm (true)
term-hook notEqualTerm (false))] .
op _=/=_ : Universal Universal -> Bool [poly (1 2) prec 51 gather (E E) special
(
id-hook EqualitySymbol
term-hook equalTerm (false)
term-hook notEqualTerm (true))] .
op true : -> Bool [ctor special (
id-hook SystemTrue)] .
op false : -> Bool [ctor special (
id-hook SystemFalse)] .
op _and_ : Bool Bool -> Bool [assoc comm prec 55 gather (e E)] .
op _or_ : Bool Bool -> Bool [assoc comm prec 59 gather (e E)] .
op _xor_ : Bool Bool -> Bool [assoc comm prec 57 gather (e E)] .
op not_ : Bool -> Bool [prec 53 gather (E)] .
op _implies_ : Bool Bool -> Bool [prec 61 gather (e E)] .
Maude> q
Bye.
%
The previous text has been copied without any editing, just like seen in the terminal. The
session was executed on a terminal with a width of 80 characters. This length is important,
since Maude takes into account this length to break lines. Note the definitions of the operators
if_then_else_fi, _==_, and _=/=_, whose definitions have been cut in multiple lines.
If we compile the output.tex file with pdflatex we get the output shown in Figure 20.1.1A
few comments on it:
• The log includes information on (most) commands and their outputs. For example, the
module being pasted is not shown, and the typing error is not included, nor the error
message.
• The first representation of the module is obtained using the show module command (see
Appendix A.12). Note that the module is shown as entered, including the terms in
statements, which are not formatted.
• The second representation of the module is produced with the show desugaredcommand
(see Appendix A.12). In this case, terms are typeset using their internal representation.
The rest of the differences come from the differences between the show module and the
show desugared commands. E.g., importations are made explicit, precedence and gath-
ering information is included, variables declarations are separated, etc.
• Different fonts are used for the text, and the use of proportional fonts (i.e., fonts with no
fixed character width) may make the text more compact in some cases. The definitions
of the operators _==_ and _=/=_, without the hooks now fit in one line.
• The declaration for if_then_else_fi still does not fit, but notice that it is the LATEX
typesetting system that now decides where to break it.
• Symbols like ->, ~>, =>, and /\ are replaced with symbols →, ⇝, ⇒, and ∧, respectively.2
1Figure 20.1 has been produced by compiling the output.tex file.
2The characters being replaced are fˆn for iterated symbols, -> for operator declarations, ˜> for partial
operator declarations, /\for conjunction in conditions, => for rules, <=? for matching, =? for unification, <- for
strategy assignment, =>1, =>+, =>*, =>!, and =># for search, and –> for assignments , ===[ rule ]===> for paths

<!-- manual page 520 -->
508 CHAPTER 20. LATEX SUPPORT
Although the look is nicer, and it is easy to read and match to the corresponding ASCII
representation, note that copying and pasting this text will produce error messages if
entered back into Maude. The ASCII option of the maude LATEX package does not replace
these symbols. The compilation of the output.tex file using the ASCII option produces
the output shown in Figure 20.2.3
20.2 The latex attribute
The latex attribute allows us to change the representation of operators and therefore terms
to be used in the LATEX log. For example, the following operator has an associated LATEX
formatting:
op integral : Expr Expr Expr Var -> Expr [latex (\int_{#1}^{#2}{#3}\,d{#4})] .
Note that the LATEX code is enclosed by parentheses rather than quotes. With this decision, we
avoid the need to use the string backslash escape convention to write \ (backslash), which is
very common in LATEX code. Thus, we write \int rather than \\int. A consequence of this is
that the parentheses outside of braces in the latex code must balance. However, parentheses
and braces appearing directly after any of
\ \left \right \big \Big \bigg \Bigg
\bigl \Bigl \biggl \Biggl \bigr \Bigr \biggr \Biggr
are considered part of a command and do not count for deciding where the LATEX code ends.
If you need to have a closing parenthesis that would look like the end of the LATEX code, you
can always enclose the code in braces; for example:
op p : Real Real -> Pair [latex ({)#1,#2(})] .
Argument positions are indicated using the #nnotation (borrowed from the TEX parameter
syntax). Arguments can be omitted, duplicated and appear out of order, and unlike in TEX,
do not stop at #9. However, Maude will complain if you specify #n for an operator with fewer
than n arguments.
The LATEX code is used for printing terms in statements and results, but not bare operators
themselves (e.g., in declarations and renamings). Terms are always typeset using inline math
mode, and code in the latex attribute should not try to change the mode.
Consider the following module GAUSSIAN in a file gaussian.maude.
fmod GAUSSIAN is
pr FLOAT * (op -_ to -_ [latex (-{#1})],
op _^_ to _^_ [latex ({#1}^{#2})]) .
sort Var Expr .
subsort Var Float < Expr .
op infinity : -> Float [latex (\infty)].
op e : -> Float .
op x : -> Var .
op gaussian : -> Expr .
op integral : Expr Expr Expr Var -> Expr [latex (\int_{#1}^{#2}{#3}\,d{#4})] .
op -_ : Expr -> Expr [ditto] .
op _^_ : Expr Expr -> Expr [ditto] .
with rule, –- label –-> for paths with label, –––> for paths without label, and ===> for arcs in search graph in
outputs.
3Whilst the text in Figure 20.1 has been produced in the normal mode, the text in Figure 20.2 was compiled
with the maude package included with the ASCII option.

<!-- manual page 521 -->
20.2. THE LATEX ATTRIBUTE 509
\||||||||||||||||||/
--- Welcome to Maude ---
/||||||||||||||||||\
Maude 3.5.1 built: Jul 16 2025 12:00:00
Copyright 1997-2025 SRI International
Wed Jul 16 20:00:00 2025
show module SIMPLE-NAT.
fmod SIMPLE-NAT is
sort Nat.
op zero : →Nat.
op s_ : Nat →Nat.
op _+_ : Nat Nat →Nat.
vars N M : Nat.
eq zero + N = N.
eq s N + M = s (N + M).
endfm
show desugared SIMPLE-NAT.
fmod SIMPLE-NAT is
including BOOL.
sort Nat.
op zero : →Nat.
op s_ : Nat →Nat [prec 15 gather (E)].
op _+_ : Nat Nat →Nat [prec 41 gather (E E)].
var N : Nat.
var M : Nat.
eq zero + N= N.
eq s N + M= s (N + M).
endfm
reduce in SIMPLE-NAT : s s zero + s s zero.
rewrites: 3 in 0 ms cpu (0 ms real) (200000 rewrites/second)
result Nat: s s s s zero
show ops BOOL.
op if_then_else_fi : Bool Universal Universal →Universal [poly (2 3 0) prec 0 gather (& & &)
special (
id-hook BranchSymbol
term-hook 1 (true)
term-hook 2 (false))].
op _==_ : Universal Universal →Bool [poly (1 2) prec 51 gather (E E) special (
id-hook EqualitySymbol
term-hook equalTerm (true)
term-hook notEqualTerm (false))].
op _=/=_ : Universal Universal →Bool [poly (1 2) prec 51 gather (E E) special (
id-hook EqualitySymbol
term-hook equalTerm (false)
term-hook notEqualTerm (true))].
op true : →Bool [ctor special (
id-hook SystemTrue)].
op false : →Bool [ctor special (
id-hook SystemFalse)].
op _and_ : Bool Bool →Bool [assoc comm prec 55 gather (e E)].
op _or_ : Bool Bool →Bool [assoc comm prec 59 gather (e E)].
op _xor_ : Bool Bool →Bool [assoc comm prec 57 gather (e E)].
op not_ : Bool →Bool [prec 53 gather (E)].
op _implies_ : Bool Bool →Bool [prec 61 gather (e E)].
Bye.
Figure 20.1: LATEX log of the interaction in which the module SIMPLE-NAT is entered, a term
is reduced and the operations of the module are shown.

<!-- manual page 522 -->
510 CHAPTER 20. LATEX SUPPORT
\||||||||||||||||||/
--- Welcome to Maude ---
/||||||||||||||||||\
Maude 3.5.1 built: Jul 16 2025 12:00:00
Copyright 1997-2025 SRI International
Wed Jul 16 20:00:00 2025
show module SIMPLE-NAT.
fmod SIMPLE-NAT is
sort Nat.
op zero : –> Nat.
op s_ : Nat –> Nat.
op _+_ : Nat Nat –> Nat.
vars N M : Nat.
eq zero + N = N.
eq s N + M = s (N + M).
endfm
show desugared SIMPLE-NAT.
fmod SIMPLE-NAT is
including BOOL.
sort Nat.
op zero : –> Nat.
op s_ : Nat –> Nat [prec 15 gather (E)].
op _+_ : Nat Nat –> Nat [prec 41 gather (E E)].
var N : Nat.
var M : Nat.
eq zero + N= N.
eq s N + M= s (N + M).
endfm
reduce in SIMPLE-NAT : s s zero + s s zero.
rewrites: 3 in 0 ms cpu (0 ms real) (200000 rewrites/second)
result Nat: s s s s zero
show ops BOOL.
op if_then_else_fi : Bool Universal Universal –> Universal [poly (2 3 0) prec 0 gather (& & &)
special (
id-hook BranchSymbol
term-hook 1 (true)
term-hook 2 (false))].
op _==_ : Universal Universal –> Bool [poly (1 2) prec 51 gather (E E) special (
id-hook EqualitySymbol
term-hook equalTerm (true)
term-hook notEqualTerm (false))].
op _=/=_ : Universal Universal –> Bool [poly (1 2) prec 51 gather (E E) special (
id-hook EqualitySymbol
term-hook equalTerm (false)
term-hook notEqualTerm (true))].
op true : –> Bool [ctor special (
id-hook SystemTrue)].
op false : –> Bool [ctor special (
id-hook SystemFalse)].
op _and_ : Bool Bool –> Bool [assoc comm prec 55 gather (e E)].
op _or_ : Bool Bool –> Bool [assoc comm prec 59 gather (e E)].
op _xor_ : Bool Bool –> Bool [assoc comm prec 57 gather (e E)].
op not_ : Bool –> Bool [prec 53 gather (E)].
op _implies_ : Bool Bool –> Bool [prec 61 gather (e E)].
Bye.
Figure 20.2: LATEX log of the interaction in which the module SIMPLE-NAT is entered, a term
is reduced and the operations of the module are shown. ASCII option on.

<!-- manual page 523 -->
20.2. THE LATEX ATTRIBUTE 511
eq gaussian = integral(- infinity, infinity, e ^ (- (x ^ 2.0)), x) .
endfm
Note the renaming of operators-_ and _^_, added just to have a latex attribute associated
to them. Although the latex attribute is to be added on the operators on sort Expr, all subsort
overloaded operators must have the same latex attribute. It is just a minor detail, but notice
that by using these latex attributes we will be seeing the minus operator close to the number
(e.g.,−2 instead of−2) and the exponentiation as we are used to (e.g., 23 instead of 2^3). We
will come back to the subtleties of the latex attribute below.
Now, consider the following interaction.
% maude -latex-log=output.tex
\||||||||||||||||||/
--- Welcome to Maude ---
/||||||||||||||||||\
Maude 3.5.1 built: Jul 16 2025 12:00:00
Copyright 1997-2025 SRI International
Wed Jul 16 20:00:00 2025
Maude> load gaussian
Maude> show module .
fmod GAUSSIAN is
protecting FLOAT * (op -_ to -_ [latex (-{#1})], op _^_ to _^_ [latex ({#1}^{
#2})]) .
sorts Var Expr .
subsorts Var Float < Expr .
op infinity : -> Float [latex (\infty)] .
op e : -> Float .
op x : -> Var .
op gaussian : -> Expr .
op integral : Expr Expr Expr Var -> Expr [latex (\int_{#1}^{#2}{#3}\,d{#4})] .
op -_ : Expr -> Expr [ditto] .
op _^_ : Expr Expr -> Expr [ditto] .
eq gaussian = integral (- infinity, infinity, e ^ (- (x ^ 2.0)), x) .
endfm
Maude> show desugared .
fmod GAUSSIAN is
including BOOL .
protecting FLOAT * (op _^_ to _^_ [latex ({#1}^{#2})], op -_ to -_ [latex (-{
#1})]) .
sorts Var Expr .
subsorts Var Float < Expr .
op -_ : Expr -> Expr [prec 15 gather (E) latex (-{#1}) special (
id-hook FloatOpSymbol (-)
op-hook floatSymbol (<Floats> : ~> FiniteFloat))] .
op _^_ : Expr Expr -> Expr [prec 41 gather (E E) latex ({#1}^{#2}) special (
id-hook FloatOpSymbol (^)
op-hook floatSymbol (<Floats> : ~> FiniteFloat))] .
op infinity : -> Float [latex (\infty)] .
op e : -> Float .
op x : -> Var .
op gaussian : -> Expr .
op integral : Expr Expr Expr Var -> Expr [latex (\int_{#1}^{#2}{#3}\,d{#4})] .
eq gaussian = integral(- infinity, infinity, e ^ - (x ^ 2.0), x) .
endfm

<!-- manual page 524 -->
512 CHAPTER 20. LATEX SUPPORT
Maude> red gaussian .
reduce in GAUSSIAN : gaussian .
rewrites: 1 in 0ms cpu (0ms real) (32258 rewrites/second)
result Expr: integral(- infinity, infinity, e ^ - (x ^ 2.0), x)
Maude> q
Bye.
%
The LATEX code in the output.tex produces the following output.4
\||||||||||||||||||/
--- Welcome to Maude ---
/||||||||||||||||||\
Maude 3.5.1 built: Jul 16 2025 12:00:00
Copyright 1997-2025 SRI International
Wed Jul 16 20:00:00 2025
show module GAUSSIAN.
fmod GAUSSIAN is
protecting FLOAT * (op -_ to -_ [latex (-{#1})],op _^_ to _^_ [latex
({#1}^{#2})]).
sorts Var Expr.
subsorts Var Float < Expr.
op infinity : →Float [latex (\infty)].
op e : →Float.
op x : →Var.
op gaussian : →Expr.
op integral : Expr Expr Expr Var →Expr [latex (\int_{#1}^{#2}{#3}\,d{#4})].
op -_ : Expr →Expr [ditto].
op _^_ : Expr Expr →Expr [ditto].
eq gaussian = integral (- infinity, infinity, e ^ (- (x ^ 2.0)), x).
endfm
show desugared GAUSSIAN.
fmod GAUSSIAN is
including BOOL.
protecting FLOAT * (op _^_ to _^_ [latex ({#1}^{#2})],op -_ to -_ [latex
(-{#1})]).
sorts Var Expr.
subsorts Var Float < Expr.
op -_ : Expr →Expr [prec 15 gather (E) latex (-{#1}) special (
id-hook FloatOpSymbol (-)
op-hook floatSymbol (<Floats> : ⇝ FiniteFloat))].
op _^_ : Expr Expr →Expr [prec 41 gather (E E) latex ({#1}^{#2}) special (
id-hook FloatOpSymbol (^)
op-hook floatSymbol (<Floats> : ⇝ FiniteFloat))].
op infinity : →Float [latex (\infty)].
op e : →Float.
4The text of the output.tex file has been produced by pasting the whole text between \begin{document}
and \end{document} in the output.tex file, and compiling it as part of the rest of the document.

<!-- manual page 525 -->
20.2. THE LATEX ATTRIBUTE 513
op x : →Var.
op gaussian : →Expr.
op integral : Expr Expr Expr Var →Expr [latex (\int_{#1}^{#2}{#3}\,d{#4})].
eq gaussian=
∞
e−x2.0 dx.
−∞
endfm
reduce in GAUSSIAN : gaussian.
rewrites: 1 in 0 ms cpu (0 ms real) (32258 rewrites/second)
result Expr:
∞
e−x2.0 dx
−∞
Bye.
Note that
• As already pointed out, not all commands are included in the log. In this case, the load
command is not reflected.
• The latex attribute is used for nicely typesetting the terms in statements in the repre-
sentation of modules generated by the show desugared commands, but not for the show
module commands.
Now, consider the following modules PAIR and NAT-PAIR modules, both included in a file
nat-pair.maude. Given the nat-pair.maude file below, the PDF corresponding to the inter-
action in Figure 20.3 is shown in Figure 20.4. Note that both the term to be reduced and the
resulting term are typeset. However, the trace is not included in the log.
fmod PAIR{X :: TRIV} is
pr NAT .
sort Pair{X} .
op p : X$Elt X$Elt -> Pair{X} [latex ((#1,#2))] .
op p : Nat Pair{X} -> X$Elt [latex ({p_{#1}#2})] .
vars N1 N2 : X$Elt .
eq p(1, p(N1, N2)) = N1 .
eq p(2, p(N1, N2)) = N2 .
endfm
fmod NAT-PAIR is
pr PAIR{Nat} .
endfm
Some additional notes:
• There is no scaling of parentheses, brackets or braces surrounding large LATEX constructs
because Maude has no idea how tall such constructs are and the automatic scaling com-
mands such as \left( and \right) do not allow line breaking between them, which
is typically required for large terms. See the use of the LATEX commands \left( and
\right) in the interaction for the PRODUCT module in Figures 20.5 and 20.6.
• When printing terms, the LATEX code for operators is blindly passed through into the
LATEX log. Furthermore no (...).sort disambiguations or parentheses for grouping are
added outside or inside of an operator with a latex attribute since it is assumed that
the user’s LATEX representation fully disambiguates any ambiguities that arise in the text
version.
• Associative operators with a latex attribute, appearing on top of a flattened argument
list, are printed as if the argument list was nested in right associative form.
• When one renames an operator with a latex attribute, the attribute is assumed to refer
to the old name and is discarded. However, a new latex attribute can be specified in the
renaming. For example, consider the following interaction.

<!-- manual page 526 -->
514 CHAPTER 20. LATEX SUPPORT
% maude -latex-log=output.tex
\||||||||||||||||||/
--- Welcome to Maude ---
/||||||||||||||||||\
Maude 3.5.1 built: Jul 16 2025 12:00:00
Copyright 1997-2025 SRI International
Wed Jul 16 20:00:00 2025
Maude> load nat-pair.maude
Maude> set trace on .
Maude> red p(1, p(1,2)) .
reduce in NAT-PAIR : p(1, p(1, 2)) .
*********** equation
eq p(1, p(N1:Nat, N2:Nat)) = N1:Nat .
N1:Nat --> 1
N2:Nat --> 2
p(1, p(1, 2))
--->
1
rewrites: 1 in 0ms cpu (0ms real) (12048 rewrites/second)
result NzNat: 1
Maude> q
Bye.
Figure 20.3: Interaction for the NAT-PAIR module.
\||||||||||||||||||/
--- Welcome to Maude ---
/||||||||||||||||||\
Maude 3.5.1 built: Jul 16 2025 12:00:00
Copyright 1997-2025 SRI International
Wed Jul 16 20:00:00 2025
reduce in NAT-PAIR : p1(1,2).
rewrites: 1 in 0 ms cpu (0 ms real) (12048 rewrites/second)
result NzNat: 1
Bye.
Figure 20.4: LATEX log of the interaction in Figure 20.3.

<!-- manual page 527 -->
20.2. THE LATEX ATTRIBUTE 515
% maude -latex-log=output.tex
\||||||||||||||||||/
--- Welcome to Maude ---
/||||||||||||||||||\
Maude 3.5.1 built: Jul 16 2025 12:00:00
Copyright 1997-2025 SRI International
Wed Jul 16 20:00:00 2025
Maude> fmod FOO is
> sort Real .
> op _+_ : Real Real -> Real [assoc comm latex (#1 \oplus #2)] .
> ops 0 1 2 3 4 : -> Real .
> endfm
Maude> red 0 + 1 + 2 + 3 + 4 .
reduce in FOO : 0 + 1 + 2 + 3 + 4 .
rewrites: 0 in 0ms cpu (0ms real) (0 rewrites/second)
result Real: 0 + 1 + 2 + 3 + 4
Maude> fmod BAR is
> inc FOO * (op _+_ to _*_) .
> endfm
Maude> red 0 * 1 * 2 * 3 * 4 .
reduce in BAR : 0 * 1 * 2 * 3 * 4 .
rewrites: 0 in 0ms cpu (0ms real) (0 rewrites/second)
result Real: 0 * 1 * 2 * 3 * 4
Maude> fmod BAZ is
> inc FOO * (op _+_ to _*_ [latex (#1 \otimes #2)]) .
> endfm
Maude> red 0 * 1 * 2 * 3 * 4 .
reduce in BAZ : 0 * 1 * 2 * 3 * 4 .
rewrites: 0 in 0ms cpu (0ms real) (0 rewrites/second)
result Real: 0 * 1 * 2 * 3 * 4
Maude> q
Bye.
%
The interaction shows several modules in which and operator is renamed without and with
the specification of a latex attribute. The log file produces the following output (in this case,
only the excerpts of the LATEX file corresponding to the reduce commands has been included).
reduce in FOO : 0 ⊕1 ⊕2 ⊕3 ⊕4.
rewrites: 0 in 0 ms cpu (0 ms real) (0 rewrites/second)
result Real: 0 ⊕1 ⊕2 ⊕3 ⊕4
reduce in BAR : 0 * 1 * 2 * 3 * 4.
rewrites: 0 in 0 ms cpu (0 ms real) (0 rewrites/second)
result Real: 0 * 1 * 2 * 3 * 4
reduce in BAZ : 0 ⊗1 ⊗2 ⊗3 ⊗4.
rewrites: 0 in 0 ms cpu (0 ms real) (0 rewrites/second)
result Real: 0 ⊗1 ⊗2 ⊗3 ⊗4
In some cases we may want to activate or deactivate the pretty printing of terms, e.g., if we
want to re-enter in Maude part or all of the text. The set print latex on/off . command
controls whether latex attributes are used for generating LATEX for terms.

<!-- manual page 528 -->
516 CHAPTER 20. LATEX SUPPORT
Consider the following example, illustrating the set print latex on/off .command and
some other possibilities in the latex attribute, like parentheses, products, etc. Given the
PRODUCT module below, the PDF corresponding to the interaction in Figure 20.5 is shown in
Figure 20.6. Note the different printing of terms in the first and second show desugared module
and reduce commands, given the use of the set print latex on/off . command between
them.
fmod PRODUCT is
pr SET{Float} * (op empty to empty [latex (\emptyset)],
op _*_ to _*_ [latex (#1 \cdot #2)]) .
op product : Set{Float} -> Float [latex (\prod_{i=\{#1\}}^{}{\bigl( i \bigr)})] .
var F : Float .
var FS : Set{Float} .
eq product((F, FS)) = F * product(FS) .
eq product(empty) = 1.0 .
endfm

<!-- manual page 529 -->
20.2. THE LATEX ATTRIBUTE 517
% maude -latex-log=output.tex
\||||||||||||||||||/
--- Welcome to Maude ---
/||||||||||||||||||\
Maude 3.5.1 built: Jul 16 2025 12:00:00
Copyright 1997-2025 SRI International
Wed Jul 16 20:00:00 2025
Maude> load product.maude
Maude> show desugared PRODUCT .
fmod PRODUCT is
including BOOL .
protecting SET{Float} * (op _*_ to _*_ [latex (#1 \cdot #2)], op empty to empty [
latex (\emptyset)]) .
op product : Set{Float} -> Float [latex (\prod_{i=\{#1\}}^{}{\bigl( i \bigr)})] .
var F : Float .
var FS : Set{Float} .
eq product((F, FS)) = F * product(FS) .
eq product(empty) = 1.0 .
endfm
Maude> red product((2.0,3.0,4.0)) .
reduce in PRODUCT : product((2.0, 3.0, 4.0)) .
rewrites: 7 in 0ms cpu (0ms real) (40000 rewrites/second)
result FiniteFloat: 2.4e+1
Maude> set print latex off .
Maude> show desugared PRODUCT .
fmod PRODUCT is
including BOOL .
protecting SET{Float} * (op _*_ to _*_ [latex (#1 \cdot #2)], op empty to empty [
latex (\emptyset)]) .
op product : Set{Float} -> Float [latex (\prod_{i=\{#1\}}^{}{\bigl( i \bigr)})] .
var F : Float .
var FS : Set{Float} .
eq product((F, FS)) = F * product(FS) .
eq product(empty) = 1.0 .
endfm
Maude> red product((2.0,3.0,4.0)) .
reduce in PRODUCT : product((2.0, 3.0, 4.0)) .
rewrites: 7 in 0ms cpu (0ms real) (205882 rewrites/second)
result FiniteFloat: 2.4e+1
Maude> q
Bye.
Figure 20.5: Interaction for the PRODUCT module.

<!-- manual page 530 -->
518 CHAPTER 20. LATEX SUPPORT
\||||||||||||||||||/
--- Welcome to Maude ---
/||||||||||||||||||\
Maude 3.5.1 built: Jul 16 2025 12:00:00
Copyright 1997-2025 SRI International
Wed Jul 16 20:00:00 2025
show desugared PRODUCT.
fmod PRODUCT is
including BOOL.
protecting SET{Float}* (op _*_ to _*_ [latex (#1 \cdot #2)],op empty to empty [latex
(\emptyset)]).
op product : Set{Float}→Float [latex (\prod_{i=\{#1\}}^{}{\bigl( i \bigr)})].
var F : Float.
var FS : Set{Float}.
eq i={F , FS} i= F·
i={FS} i.
eq i={∅} i = 1.0.
endfm
reduce in PRODUCT : i={2.0, 3.0, 4.0} i.
rewrites: 7 in 0 ms cpu (0 ms real) (40000 rewrites/second)
result FiniteFloat: 2.4e+ 1
show desugared PRODUCT.
fmod PRODUCT is
including BOOL.
protecting SET{Float}* (op _*_ to _*_ [latex (#1 \cdot #2)],op empty to empty [latex
(\emptyset)]).
op product : Set{Float}→Float [latex (\prod_{i=\{#1\}}^{}{\bigl( i \bigr)})].
var F : Float.
var FS : Set{Float}.
eq product((F, FS)) = F * product(FS).
eq product(empty) = 1.0.
endfm
reduce in PRODUCT : product((2.0, 3.0, 4.0)).
rewrites: 7 in 0 ms cpu (0 ms real) (205882 rewrites/second)
result FiniteFloat: 2.4e+ 1
Bye.
Figure 20.6: LATEX log of the interaction in Figure 20.5. It illustrates the use of the set print
latex off command.

