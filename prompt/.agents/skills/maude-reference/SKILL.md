---
name: maude-reference
description: The Maude 3.5.1 manual split into one file per chapter. Use when a pipeline step's Input lists this skill and you need Maude syntax or semantics.
---

[Maude Reference]
The Maude 3.5.1 manual, split into one file per chapter under references/. The text of each chapter is the text of the manual; the bibliography and the indexes are omitted.

# How to use
1. Find the chapter you need in the index below, using its title and section titles.
2. Read only the chapter files you need. Each page starts with a `<!-- manual page N -->` line.

# Index

- Chapter 1: Introduction — references/01-introduction.md (pages 13-30)
  - Sections: Simplicity, expressiveness, and performance; The logical foundations of Maude; Programming, specification, and verification; A high-performance logical framework; Manual structure; The Maude book
- Chapter 2: Using Maude — references/02-using-maude.md (pages 31-36)
  - Sections: Getting Maude; Running Maude; Getting support and more information; Reporting bugs in Maude
- Chapter 3: Syntax and Basic Parsing — references/03-syntax-and-basic-parsing.md (pages 37-54)
  - Sections: Identifiers; Modules; Sorts and subsorts; Operator declarations; Kinds; Operator overloading; Variables; Terms and preregularity; Parsing
- Chapter 4: Functional Modules — references/04-functional-modules.md (pages 55-96)
  - Sections: Unconditional equations; Unconditional memberships; Conditional equations and memberships; Operator attributes; Statement attributes; Admissible functional modules; Matching and equational simplification; More on matching and simplification modulo; The reduce, match, trace, and show commands
- Chapter 5: System Modules — references/05-system-modules.md (pages 97-112)
  - Sections: Unconditional rules; Conditional rules; Admissible system modules; The rewrite, frewrite, and search commands
- Chapter 6: Object-Oriented Modules — references/06-object-oriented-modules.md (pages 113-142)
  - Sections: Objects, messages and configurations; Example: a rent-a-car store; Object-message fair rewriting; Object-based programming; From object-oriented modules to system modules
- Chapter 7: Module Operations — references/07-module-operations.md (pages 143-196)
  - Sections: Module importation; The summation module expression; Module renaming; Parameterized programming; Example: data agents; Example: extended rent-a-car store
- Chapter 8: Predefined Data Modules — references/08-predefined-data-modules.md (pages 197-268)
  - Sections: Boolean values; Initial equality predicate; Natural numbers; Random numbers and counters; Integer numbers; Machine integers; Rational numbers; Floating-point numbers; Strings; String and number conversions; Quoted identifiers; Conversions between strings and lists of quoted ids.; Basic theories and standard views; Containers: lists and sets; Maps and arrays; A linear Diophantine equation solver; Predefined parameterized views
- Chapter 9: External Objects and IO — references/09-external-objects-and-io.md (pages 269-308)
  - Sections: Standard streams; File I/O; Directory API; Sockets; Processes; Time API; Pseudo-random number generator objects; Control-C on external events
- Chapter 10: Strategy Language — references/10-strategy-language.md (pages 309-330)
  - Sections: The strategy language; Strategy modules; Parameterization in strategy modules; Strategy search and the dsrewrite command
- Chapter 11: Model Checking Invariants Through Search — references/11-model-checking-invariants-through-search.md (pages 331-340)
  - Sections: Invariants; Model checking of invariants; Bounded model checking of invariants; Verifying infinite-state systems through abstractions
- Chapter 12: LTL Model Checking — references/12-ltl-model-checking.md (pages 341-364)
  - Sections: LTL formulas and the LTL module; Associating Kripke structures to rewrite theories; LTL model checking; The LTL satisfiability and tautology checker; Model checking a round-robin scheduling algorithm; LTL+ formulas and the LTL+ model checker
- Chapter 13: Unification — references/13-unification.md (pages 365-382)
  - Sections: Introduction; Order-sorted unification; Theories currently supported; The unify command
- Chapter 14: Variants and Variant Unification — references/14-variants-and-variant-unification.md (pages 383-398)
  - Sections: Introduction; Term variants; Theories currently supported; The get variants command; Variant generation with irreducibility constraints; Incremental variant generation; Variant generation in incomplete unification examples; Variant-based equational order-sorted unification; The variant unify command; Variant-based unification with irreducibility constraints; Incremental variant unification; Variant unification in incomplete unification examples; The variant match command
- Chapter 15: Narrowing — references/15-narrowing.md (pages 399-424)
  - Sections: Introduction; Applications; Completeness of narrowing; Narrowing with simplification; Theories supported for narrowing reachability; The vu-narrow command; Narrowing with extra variables in righthand sides of rules
- Chapter 16: SMT Solving — references/16-smt-solving.md (pages 425-430)
  - Sections: Boolean formulas; Formulas using integer linear arithmetic; Formulas using rational linear arithmetic; Formulas using rational and integer linear arithmetic; Satisfiability of formulas
- Chapter 17: Reflection, Metalevel Computation, and Internal Strategies — references/17-reflection-metalevel-computation-and-internal-strategies.md (pages 431-488)
  - Sections: Reflection and metalevel computation; The META-TERM module; The META-STRATEGY module: Metarepresenting the strategy language; The META-MODULE module: Metarepresenting modules; The META-VIEW module: Metarepresenting views; The META-LEVEL module: Metalevel operations; Internal strategies
- Chapter 18: User Interfaces and Metalanguage Applications — references/18-user-interfaces-and-metalanguage-applications.md (pages 489-504)
  - Sections: User interfaces; The interaction with the system; Tokens, bubbles, and metaparsing; The LOOP-MODE module
- Chapter 19: Meta-interpreters — references/19-meta-interpreters.md (pages 505-516)
  - Sections: Maude meta-interpreters; A Russian dolls example; An execution environment for Mini-Maude
- Chapter 20: LaTeX Support — references/20-latex-support.md (pages 517-530)
  - Sections: Some basic first examples; The latex attribute
- Chapter 21: Debugging and Troubleshooting — references/21-debugging-and-troubleshooting.md (pages 531-570)
  - Sections: Debugging approaches; Debugging strategy executions; Traps and known problems
- Chapter 22: Complete List of Maude Commands — references/22-complete-list-of-maude-commands.md (pages 571-586)
  - Sections: Command line flags; Rewriting commands; Matching commands; Searching commands; Strategic rewriting commands; Unification, variants, and narrowing commands; SMT commands; Tracing commands; Print attribute commands; Print option commands; Show option commands; Show commands; Profiler commands; Debugger commands; Miscellaneous commands; System level commands
- Chapter 23: Maude Grammar — references/23-maude-grammar.md (pages 587-596)
  - Sections: The grammar; Synonyms; Lexical and other Issues
