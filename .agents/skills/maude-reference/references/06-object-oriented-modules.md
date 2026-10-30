# Chapter 6: Object-Oriented Modules (manual pages 113-142)

<!-- manual page 113 -->
Chapter 6
Object-Oriented Modules
Distributed systems can be naturally modeled in Maude as multisets of entities, loosely coupled
bysomesuitablecommunicationmechanism. Animportantexampleisobject-baseddistributed
systems in which the entities are objects, each with a unique identity, and the communication
mechanism is message passing.
Concurrent object-oriented systems can be defined by means of object-oriented modules—
introduced by the keyword omod...endom—with a syntax that assumes acquaintance with
the basic object-oriented entities, such as objects, messages and configurations, and supports
linguistic distinctions appropriate for the object-oriented case.
As we will see in Section 6.4, we may have specifications of object-based systems in sys-
tem modules. However, although Maude’s system modules are sufficient for specifying object-
oriented systems, there are important conceptual advantages provided by the syntax for object-
oriented modules. Such syntax allows the user to think and express his/her thoughts in object-
oriented terms whenever such a viewpoint seems best suited for the problem at hand. Those
conceptual advantages would be partially lost if only system modules were provided.
Object-oriented modules are however just syntactic sugar: they are internally transformed
into system modules for execution purposes (see Section 6.5). All object-oriented modules
implicitly include the CONFIGURATION module (see Section 6.1), and thus assume the syntax it
provides. The module CONFIGURATION defines the basic concepts of concurrent object systems;
among others, and besides the Configuration sort, it includes the declarations of sorts Oid of
object identifiers, Cid of class identifiers, Object for objects, and Msg for messages.
6.1 Objects, messages and configurations
Maudesupportsthemodelingofobject-basedsystemsbyprovidingapredefinedCONFIGURATION
module. This module declares sorts representing the essential concepts of object, message, and
configuration, along with a notation for object syntax that serves as a common language for
specifying object-based systems. In addition, there is an object-message fair rewriting strategy
that is well suited for executing object system configurations (see Section 6.3). Object-oriented
modules will implicitly import the CONFIGURATION module, and use its definitions to build
object-oriented systems. Indeed, as we will explain in Section 6.5, Maude desugars object-
oriented modules into system modules, built on the CONFIGURATION module as any user can
directly do.
The predefined module CONFIGURATION in the file prelude.maude provides sorts and con-
structors for modeling object-based systems.
101

<!-- manual page 114 -->
102 CHAPTER 6. OBJECT-ORIENTED MODULES
mod CONFIGURATION is
sorts Attribute AttributeSet .
subsort Attribute < AttributeSet .
op none : -> AttributeSet [ctor] .
op _,_ : AttributeSet AttributeSet -> AttributeSet
[ctor assoc comm id: none format (d d s d)] .
sorts Oid Cid Object Msg Portal Configuration .
subsort Object Msg Portal < Configuration .
op <_:_|_> : Oid Cid AttributeSet -> Object
[ctor object
special (id-hook ObjectConstructorSymbol
op-hook attributeSetSymbol (_,_ : AttributeSet AttributeSet ~> AttributeSet))] .
op none : -> Configuration [ctor] .
op __ : Configuration Configuration -> Configuration
[ctor config assoc comm id: none] .
op <> : -> Portal [ctor portal] .
op getClass : Object -> Cid .
eq getClass(< O:Oid : C:Cid | A:AttributeSet >) = C:Cid .
endm
The basic sorts needed to describe an object system are: Object, Msg (messages), and
Configuration. For the different elements involved in their construction, four additional sorts
are declared: Oid (object identifiers), Cid (class identifiers), Attribute (a named element of
an object’s state), and AttributeSet (multisets of attributes).
An object in a given state is represented as a term of the form
< O : C | ⟨att-1⟩, ..., ⟨att-n⟩ >
for O a term of sort Oid and C a term of sort Cid. Each attribute of sort Attribute consists of
a name (attribute identifier), followed by a colon ‘:’ (with white space both before and after),
followed by its value, which must have a given type. Therefore, the syntax for objects is
< O : C | a1 : v1, ..., an : vn >
where O is the object’s name or identifier, C is its class identifier, the ai’s are the names of
the object’s attribute identifiers, and the vi’s are the corresponding values, for i = 1 ...n. In
particular, an object with no attributes can be represented as
< O : C | >
or
< O : C | none >
Messages do not have a fixed syntactic form; their syntax is instead defined by the user
for each application. The only assumption made by the system is that the first argument of
a message is the identifier of its destination object. This requirement is key for the correct
operation of the object-message fair rewriting strategy (see Section 6.3).
Theconcurrentstateofanobject-orientedsystemisthenamultisetofobjectsandmessages,
of sort Configuration, with multiset union described with empty syntax__, and with assoc,
comm, and id: none as operator attributes. The empty configuration is represented by the
constant none. The attribute config declares that configurations constructed with__ support
the special object-message fair rewriting behavior (see Section 6.3).
A typical configuration will have the form

<!-- manual page 115 -->
6.1. OBJECTS, MESSAGES AND CONFIGURATIONS 103
⟨Ob-1⟩... ⟨Ob-k⟩ ⟨Mes-1⟩... ⟨Mes-n⟩
where ⟨Ob-1⟩, ..., ⟨Ob-k⟩are objects, ⟨Mes-1⟩, ..., ⟨Mes-n⟩are messages, and the order is
immaterial.
An important consideration about object systems is that object names must be unique.
More precisely there should not be two objects with the same identifier in the same configura-
tion. There might be repeated identifiers in nested configurations, or in separate configurations,
but not in the same configuration. This property is key for message delivery.
In general, a rewrite rule for an object system has the form
rl ⟨Ob-1⟩... ⟨Ob-k⟩ ⟨Mes-1⟩... ⟨Mes-n⟩
=> ⟨Ob’-1⟩... ⟨Ob’-j⟩ ⟨Ob-k+1⟩... ⟨Ob-m⟩ ⟨Mes’-1⟩... ⟨Mes’-p⟩.
where ⟨Ob’-1⟩, ..., ⟨Ob’-j⟩are updated versions of ⟨Ob-1⟩, ..., ⟨Ob-j⟩for j ≤k, ⟨Ob-k+1⟩,
..., ⟨Ob-m⟩are newly created objects, and ⟨Mes’-1⟩, ..., ⟨Mes’-p⟩are new messages. An
important special case are rules with a single object and at most one message on the lefthand
side. These are called asynchronous rules. They directly model asynchronous distributed
interactions. Rules involving multiple objects are called synchronous; they are used to model
higher-level communication abstractions.
The operation getClass takes an object as argument and returns its actual class. See
examples below. This operation will be particularly useful when combined with the inheritance
relation (e.g., see the use of the getClass operation in the example in Section 7.6).
TheremainderoftheCONFIGURATIONmoduleprovidesdefinitionsrelatedtoexternalobjects
(see Chapter 9).
6.1.1 Classes
Classes are defined with the keyword class, followed by the name of the class, followed by
a bar ‘|’, followed by a list of attribute declarations separated by commas. Each attribute
declaration has the form a: S, where ais an attribute identifier and S is the sort in which the
values of the attribute identifier range. That is, class declarations have the form
class C | a1 : ⟨Sort-1⟩, ..., an : ⟨Sort-n⟩.
We can declare classes without attributes using syntax
class C .
Asanexampleofclassdeclaration, theclassAccountofbankaccountobjectswithabalance
attribute, can be declared as follows:
class Account | bal : Int .
With this declaration, we are able to write account objects as < A : Account | bal : N >,
with A a term of sort Oid and N a term of sort Int. An attribute name is any single token name
that does not contain the underline character.
As another example, a class Person, with a name, an age, and a bank account as attributes
can then be declared as follows:
class Person | name : String, age : Nat, account : Oid .
In this case, a person has a reference to his/her account in an account attribute of sort Oid.
For example, we can write a configuration
< A : Account | bal : 2000 >
< P : Person | name : "Alice", age : 24, account : A >

<!-- manual page 116 -->
104 CHAPTER 6. OBJECT-ORIENTED MODULES
with A and P terms of sort Oid.
As we saw in the previous section, all object-oriented modules have an operation getClass,
defined in the CONFIGURATION module, that takes an object as argument and returns its actual
class. Thus, for example,
getClass(< A-002 : Account | bal : 1000 >)
returns the class identifier Account.
Class names have the same form as sort names (see Section 3.3). In particular, as we will
explain in Section 7.4.3, class names may be parameterized in a way completely similar to
parameterized sort names.
6.1.2 Messages
The syntax for message declarations is similar to the syntax for the declaration of operators,
but using msg and msgs instead of op and ops, and having as result the sort Msg or a subsort of
it. Thus, msg is used to declare a single message, and msgs may be used for declaring multiple
messages. The user can introduce subsorts of the predefined sort Msg, so that it is possible to
declare messages of different types. This may be useful for restricting the kind of messages that
could be received by a particular type of object. As in the case of operators, messages can be
overloaded and can be declared with operator attributes.
In the account example, the three kinds of messages involving accounts are credit, debit,
and from_to_transfer_, whose user-definable syntax is introduced in the following declara-
tions:
msgs credit debit : Oid Nat -> Msg .
msg from_to_transfer_ : Oid Oid Nat -> Msg .
Notice the use of the Oid sort for specifying the addressee of a message, as in credit and
debit, and for specifying the objects involved in a message, e.g., the source and the target
accounts of an account transfer in the from_to_transfer_ message. As already pointed out,
it is assumed that the message’s destination object is the first argument mentioned in the
message declaration. This convention is needed by the object-message fair rewriting strategy
(see Section 6.3). The behavior associated with the messages is then specified by rewrite rules
in a declarative way (see Section 6.1.4). Given object identifiers Smith and A-002, the following
term may represent a configuration with a person, his account, and a credit message sent to
it.
< Smith : Person | name : "John", age : 34, account : A-002 >
< A-002 : Account | bal : 1000 >
credit(A-002, 100)
6.1.3 Class inheritance
Class inheritance is directly supported by Maude’s order-sorted type structure. Since class
names are a particular case of sort names, a subclass declaration C < C’ in an object-oriented
module is just a particular case of a subsort declaration C < C’. The effect of a subclass
declaration is that the attributes, messages, and rules of all the superclasses, together with the
newly defined attributes, messages, and rules of the subclass, characterize the structure and
behavior of the objects in the subclass.
For example, we can define an object-oriented module SAVING-ACCOUNT of saving accounts
introducing a subclass SavingAccount of Account with a new attribute rate recording the
interest rate of the account.

<!-- manual page 117 -->
6.1. OBJECTS, MESSAGES AND CONFIGURATIONS 105
class SavingAccount | rate : Float .
subclass SavingAccount < Account .
In this example there is only one class immediately above SavingAccount, namely, Account.
In general, however, a class C may be defined as a subclass of several classes D1,...,Dk, i.e.,
multiple inheritance is supported. There is no restriction on the use of attributes with the same
names. Of course, having classes with attributes with the same names may be a bad idea, but
it is the responsability of the user to make it work. This may be aggravated with inheritance,
since we may be adding attributes with the same name of an attribute in a superclass. If a class
inherits from two different superclasses that share an attribute but with different associated
sorts, then both attributes are inherited in the subclass. In summary, a class inherits all the
attributes, messages, and rules from all its superclasses. An object in the subclass behaves
exactly as any object in any of the superclasses, but it may exhibit additional behavior due to
the introduction of new attributes, messages, and rules in the subclass.
Objects in the class SavingAccount will have an attribute bal and can receive messages
debiting, crediting and transferring funds exactly as any other object in the class Account. For
example, the following object is a valid instance of class SavingAccount.
< A-002 : SavingAccount | bal : 5000, rate : 3.0 >
As for subsort relationships, we can declare multiple subclass relationships in the same
declaration. Thus, given classes A, ..., H, we can have a declaration of subclasses such as
subclasses A B C < D E < F G H .
6.1.4 Object-oriented statements
The behavior associated with messages is specified by rewrite rules in a declarative way. For
example, the semantics of the credit, debit, and from_to_transfer_ messages declared in
Section 6.1.2 may be given as follows:
vars A B : Oid .
var M : Nat .
vars N N’ : Int .
rl [credit] :
credit(A, M)
< A : Account | bal : N >
=> < A : Account | bal : N + M > .
crl [debit] :
debit(A, M)
< A : Account | bal : N >
=> < A : Account | bal : N - M >
if N >= M .
crl [transfer] :
(from A to B transfer M)
< A : Account | bal : N >
< B : Account | bal : N’ >
=> < A : Account | bal : N - M >
< B : Account | bal : N’ + M >
if N >= M .

<!-- manual page 118 -->
106 CHAPTER 6. OBJECT-ORIENTED MODULES
Notethatthemultisetstructureoftheconfigurationprovidesthetop-leveldistributedstruc-
ture of the system and allows concurrent application of the rules. The reader is referred to [93]
for a detailed explanation of the logical semantics of the object-oriented model of computation
supported by Maude.
ThefollowingmoduleACCOUNTcontainsallthedeclarationsabovedefiningtheclassAccount.
Note that Qid is declared as a subsort of Oid, making any quoted identifier a valid object iden-
tifier.
omod ACCOUNT is
protecting QID .
protecting INT .
subsort Qid < Oid .
class Account | bal : Int .
msgs credit debit : Oid Int -> Msg .
msg from_to_transfer_ : Oid Oid Int -> Msg .
vars A B : Oid .
var M : Nat .
vars N N’ : Int .
rl [credit] :
credit(A, M)
< A : Account | bal : N >
=> < A : Account | bal : N + M > .
crl [debit] :
debit(A, M)
< A : Account | bal : N >
=> < A : Account | bal : N - M >
if N >= M .
crl [transfer] :
(from A to B transfer M)
< A : Account | bal : N >
< B : Account | bal : N’ >
=> < A : Account | bal : N - M >
< B : Account | bal : N’ + M >
if N >= M .
endom
We can now rewrite a simple configuration consisting of an account and a message in the
module ACCOUNT as follows:
Maude> rew < ’A-06238 : Account | bal : 2000 >
debit(’A-06238, 1000) .
result Object: < ’A-06238 : Account | bal : 1000 >
The application of the debit rule on this configuration models the reception of the debit
message by the Account object ’A-06238. Upon the reception of the message, the balance of
the object changes as a result.
In the rules in the ACCOUNT module, all occurrences of objects include all their attributes.
However, in object-oriented modules, it is possible not to mention in a given statement those
attributes of an object that are not relevant for that statement. For example, the attributes
mentioned only on the lefthand side of a rule are preserved unchanged, the original values of

<!-- manual page 119 -->
6.1. OBJECTS, MESSAGES AND CONFIGURATIONS 107
attributes mentioned only on the righthand side do not matter, and all attributes not explicitly
mentioned are left unchanged.
To illustrate that not all attributes need to be cited in rules in object-oriented modules, let
us consider the following simple example. A message for changing the age of a person defined
by the class Person (introduced in Section 6.1.1) may be defined as follows:
msg to_: new age_ : Oid Nat -> Msg .
var A : Nat .
var O : Oid .
rl [change-age] :
< O : Person | >
to O : new age A
=> < O : Person | age : A > .
The attributes name and account, which are not mentioned in this rule, are not changed
when applying the rule. The value of the age attribute is replaced by the given new age,
independently of its previous value.
These rules, as any other statement defining the behavior of classes, will be inherited by
their subclasses. In this case, inheritance means that these statements, will be applicable on
instances of subclasses. Although the details will be introduced in the following sections, let us
show an example.
The following module contains the declarations for the class SavingAccount.
omod SAVING-ACCOUNT is
including ACCOUNT .
protecting FLOAT .
class SavingAccount | rate : Float .
subclass SavingAccount < Account .
endom
Note that the SAVING-ACCOUNT module does not contain any rule. Specifically, we leave
unspecified the rules for changing interest rates, or for computing and crediting the interest of
an account according to its rate.1 However, notice that the class SavingAccount inherits the
attributes and the behavior of the Account class. For example, we now rewrite a configuration
in the SAVING-ACCOUNT module, obtaining the following result.
Maude> rew < ’A-73728 : SavingAccount | bal : 5000, rate : 3.0 >
< ’A-06238 : Account | bal : 2000 >
< ’A-28381 : SavingAccount | bal : 9000, rate : 3.0 >
debit(’A-06238, 1000)
credit(’A-73728, 1300)
credit(’A-28381, 200) .
result Configuration:
< ’A-06238 : Account | bal : 1000 >
< ’A-28381 : SavingAccount | bal : 9200, rate : 3.0 >
< ’A-73728 : SavingAccount | bal : 6300, rate : 3.0 >
Note that Account and SavingAccount objects respond similarly to the rules specified.
1The proper specification of crediting the interest of an account should require the introduction of a real-time
attribute in account objects. The interested reader may look at [105, 107] for a general methodology to specify
real-time systems, including object-oriented ones, in rewriting logic.

<!-- manual page 120 -->
108 CHAPTER 6. OBJECT-ORIENTED MODULES
AswewillseeindetailinSection6.5, statementsinobject-orientedmodulesaretransformed
so that the intended behavior is obtained. For example, the exact same behavior specified for
the Person class above could be expressed as follows:
vars A A’ : Nat .
var O : Oid .
var C : Person .
var Atts : AttributeSet .
rl [change-age] :
< O : C | age : A’, Atts >
to O : new age A
=> < O : C | age : A, Atts > .
Note that by using a variable C of type Person, which is a subsort of sort Cid, the rule may
be applied to objects in subclasses of Person. Even though the value of the age attribute
when the rule is applied is not relevant, to replace its value, it must appear in the lefthand side
of the rule. Finally, notice the Atts attribute to take all other attributes that objects of the
class have, which are left unchanged. This rule will be applied on objects of subclasses having
additional attributes as well. Additional examples are presented, e.g., in Sections 6.2 and 7.6.
The transformation performed on the statements of object-oriented modules is explained
in detail in Section 6.5. Although quite natural, it is important to keep in mind that the
transformation is not carried out on all statements. Since, e.g., a rule may have in its right-
hand side objects that are not in its left-hand side, replicated objects may show up in its
right-hand side, the class of an object may change, and other situations in which no specific
behavior is to be enforced, the user may choose to identify statements not to be transformed
with the dnt (do not transform) attribute. Indeed, the attribute is required in some cases.
Without it, Maude would produce advisories in these and other cases.
In summary, in order for a statement to be transformed, there are strict requirements
governing the occurrence of objects. If a candidate statement does not meet these requirements,
advisories will be produced and the statement will be placed in the resulting system module
untransformed. Before enumerating these restrictions, let us consider an example. If we add a
rule changing the class of an object, we will get advisories indicating the situation. If we load
the module
omod WRONG-CHANGING-SAVING-ACCOUNT is
including SAVING-ACCOUNT .
var O : Oid .
rl < O : Account | > => < O : SavingAccount | > .
endom
we get the following advisories:
Advisory: <standard input>, line 47 (omod WRONG-CHANGING-SAVING-ACCOUNT):
class argument SavingAccount in object instance < O : SavingAccount | none >
differing from class argument Account in lefthand side object instance
< O : Account | none > disables object completion.
Advisory: <standard input>, line 47 (omod WRONG-CHANGING-SAVING-ACCOUNT):
transformation not performed for:
rl < O : Account | none > => < O : SavingAccount | none > .
With these advisories Maude is pointing out that there is something problematic in the rules,
and that they will be left as provided by the user, without any transformation being applied on
them. Note that we can understand the rule in different ways. Do we want it to be applicable
only on objects of class Account or also to objects in subclasses? The subclass SavingAccount

<!-- manual page 121 -->
6.1. OBJECTS, MESSAGES AND CONFIGURATIONS 109
may have some additional attributes, what do we want to do with them? Without them objects
will not be valid instances of that class, but Maude cannot add these attributes. Therefore, to
eliminate these advisories, we can either change the rule as we wish or add a dnt attribute to
force Maude to take it as such.
In other cases, many other questions may arise. In this case, possibly the sensible thing to
do is to allow the change only to objects of the Account class, not to objects in subclasses,
since they may already have a rate attribute, or have a different behavior that may conflict
with this one.
var O : Oid .
var Atts : AttributeSet .
rl < O : Account | Atts >
=> < O : SavingAccount | rate : 3.0, Atts >
[dnt] .
A statement (mb/cmb/eq/ceq/rl/crl) that contains at least one occurrence of the object
constructoroperator(<_:_|_>)isacandidatefortransformationunlessithasthedntattribute.
The dnt attribute is ignored outside of object-oriented modules, and it disappears during trans-
lation to a system module—it is simply a flag that instructs the desugaring code to pass the
statement into the system module without further consideration.
Before introducing the restrictions on the statements to be transformed, let us introduce
some terminology:
• Within a candidate statement, two subterms with the object constructor on top, say
< t1 : t2 | t3 > and < t′
1 : t′
2 | t′
3 >, are considered to be occurrences of the same
object named t′′
1 if t′′
1 is the theory normal form of both t1 and t′
1.
• The term parts of a statement are of four kinds:
pattern: the lefthand side of the statement,
subject: the righthand side of an equation or rule,
condition-pattern: the lefthand side of an assignment condition fragment or the right-
hand side of a rule condition fragment, and
condition-subject: any other part of a condition fragment.
• If an object appears in the righthand side of a statement but not in its lefthand side, then
t is called a “new object.” If the object appears in both sides, possibly multiple times in
the righthand side, then t is called an “existing object.”
Then, the conditions any statement must satisfy in order to be transformed are:
• No objects must occur in a condition-pattern.
• An object cannot occur more than one time in the lefthand side of a statement.
• The class argument of each occurrence of an existing object must be identical and can
only be either a class constant or a variable of a class sort.
• Theattributeargumentofanexistingobjectcaneitherbea1 oranattributeseta1, ..., an
under the _,_ AttributeSet operator. Here each ai must be one of (i) the constant none,
(ii) a term headed by an attribute operator, or (iii) a variable X of sort AttributeSet.
Only one such variable X is allowed in an attribute argument, and if such a variable
appears in the attribute argument of some occurrence of an existing object E, it must
appear in the attribute argument of every occurrence of that object E. Two terms ai and
aj in the same attribute argument may not be headed by the same attribute operator.
Occurrences of existing objects are checked and may be transformed. New objects can
occur in the subject or condition-subject parts and are neither analyzed nor transformed. The
rationale is that such occurrences might have arbitrary code as second and third arguments to
choose the class and attributes of a newly created object.

<!-- manual page 122 -->
110 CHAPTER 6. OBJECT-ORIENTED MODULES
Let us conclude this section with the principles that have guided the design of the transfor-
mation:
• Transforming a statement in a way that surprises the user is worse than not providing a
transformation.
• Transformations should be all or nothing at the statement level—transforming some oc-
currences of an object but not others will likely lead to confusion.
• Transformations should work for non-executable statements—it is reasonable to have a
nonexec rule transformed and later use it at the metalevel or with the strategy language.
Please, find details on the transformation in Section 6.5.
6.1.5 Search on object-oriented modules
In the previous section, we have seen several examples on how to rewrite configurations of
objects and messages. We can also search over configurations. For example, we can look for
final states having objects of class Account with balance greater than or equal to 1000 from a
given initial configuration with the following command:
Maude> search in SAVING-ACCOUNT :
< ’A-73728 : SavingAccount | bal : 5000, rate : 3.0 >
< ’A-06238 : Account | bal : 2000 >
< ’A-28381 : SavingAccount | bal : 9000, rate : 3.0 >
debit(’A-06238, 1000)
credit(’A-73728, 1300)
credit(’A-28381, 200)
=>! C:Configuration
< O:Oid : Account | bal : N:Nat >
such that N:Nat >= 1000 .
Solution 1 (state 7)
states: 8 rewrites: 29 in 0ms cpu (0ms real) (72319 rewrites/second)
C:Configuration --> < ’A-28381 : SavingAccount | bal : 9200, rate : 3.0 >
< ’A-73728 : SavingAccount | bal : 6300, rate : 3.0 >
O:Oid --> ’A-06238
N:Nat --> 1000
No more solutions.
states: 8 rewrites: 29 in 0ms cpu (0ms real) (53505 rewrites/second)
Note that the object in the search pattern explicitly specifies an object of class Account,
and only one final state can be reached with an object of this class that satisfies the given
condition. If we want to search for objects of class Account or any of its subclasses, we may
write the following command.
Maude> search in SAVING-ACCOUNT :
< ’A-73728 : SavingAccount | bal : 5000, rate : 3.0 >
< ’A-06238 : Account | bal : 2000 >
< ’A-28381 : SavingAccount | bal : 9000, rate : 3.0 >
debit(’A-06238, 1000)
credit(’A-73728, 1300)
credit(’A-28381, 200)
=>! C:Configuration
< O:Oid : C:Account | bal : N:Nat, Atts:AttributeSet >
such that N:Nat >= 1000 .

<!-- manual page 123 -->
6.2. EXAMPLE: A RENT-A-CAR STORE 111
Solution 1 (state 7)
states: 8 rewrites: 29 in 0ms cpu (0ms real) (113725 rewrites/second)
C:Configuration --> < ’A-28381 : SavingAccount | bal : 9200, rate : 3.0 >
< ’A-73728 : SavingAccount | bal : 6300, rate : 3.0 >
O:Oid --> ’A-06238
C:Account --> Account
Atts:AttributeSet --> (none).AttributeSet
N:Nat --> 1000
Solution 2 (state 7)
states: 8 rewrites: 30 in 0ms cpu (0ms real) (88235 rewrites/second)
C:Configuration --> < ’A-06238 : Account | bal : 1000 >
< ’A-73728 : SavingAccount | bal : 6300, rate : 3.0 >
O:Oid --> ’A-28381
C:Account --> SavingAccount
Atts:AttributeSet --> rate : 3.0
N:Nat --> 9200
Solution 3 (state 7)
states: 8 rewrites: 31 in 0ms cpu (0ms real) (76923 rewrites/second)
C:Configuration --> < ’A-06238 : Account | bal : 1000 >
< ’A-28381 : SavingAccount | bal : 9200, rate : 3.0 >
O:Oid --> ’A-73728
C:Account --> SavingAccount
Atts:AttributeSet --> rate : 3.0
N:Nat --> 6300
No more solutions.
states: 8 rewrites: 31 in 0ms cpu (0ms real) (67099 rewrites/second)
In this case, we obtain three solutions, since any of the three objects in the configuration will
satisfy the condition in the final state. Please, note that the object in the search pattern has
a variable C:Account as class name, to match the Account class or any subclass of it, and a
varible Atts:AttributeSet to gather the rest of the attributes of the object. Objects in the
search pattern are left as given, and therefore, these variables are necessary if objects in any
subclass are to be found. Note that in this way one can decide what exactly one wants to search
for.
6.2 Example: a rent-a-car store
In order to further illustrate Maude’s object-oriented features, we specify a simple rent-a-car
store example. Several rules in this specification have variables in their righthand sides or
conditions not present in their lefthand sides; therefore, these rules are not directly executable
by the rewrite engine and are declared as non-executable (with the nonexec attribute). In
order to run the object-oriented system, we will have to use strategies; a possible such strategy
will be presented in Section 10.1.1.
The regulations of the system, especially those that govern the rental processes, are the
following:
1. Cars are rented for a specific number of days, after which they should be returned.
2. A car can be rented only if it is available.
3. No credit is allowed; customers must pay cash.
4. Customers must make a deposit at pick-up time of the estimated rental charges.

<!-- manual page 124 -->
112 CHAPTER 6. OBJECT-ORIENTED MODULES
5. Rental charges depend on the car class. There are three categories: economy, mid-size,
and full-size cars.
6. When a rented car is returned, the deposit is used to pay the rental charges.
7. If a car is returned before the due date, the customer is charged only for the number of
days the car has been used. The rest of the deposit is reimbursed to the customer.
8. Customers who return a rented car after its due date are charged for all the days the car
has been used, with an additional 20% charge for each day after the due date.
9. Failure to return the car on time or to pay a debt may result in the suspension of renting
privileges.
Let us begin with the static aspects of this system, i.e., with its structure. We can identify
three main classes, namely the store, customer, and car classes. There are three kinds of cars:
economy, mid-size, and full-size cars.
Customers may rent cars. This relationship may be represented by a Rental class which, in
addition to references to the objects involved in the relationship, has some extra attributes. The
system also requires some control over time, which we get with a class representing calendars
that provides the current date and simulates the passage of time.
The Customer class has three attributes, namely, suspended, cash, and debt, to keep track
of, respectively, whethersheissuspendedornot, theamountofcashthatthecustomercurrently
has, and her debt with the store. Such a class is defined by the following Maude declaration:
class Customer | cash : Nat, debt : Nat, suspended : Bool .
The attribute available of the Car class indicates whether the car is currently available or
not, and rate records the daily rental rate. We model the different types of cars for rent by
three different subclasses, namely, EconomyCar, MidSizeCar and FullSizeCar.
class Car | available : Bool, rate : Nat .
class EconomyCar .
class MidSizeCar .
class FullSizeCar .
subclasses EconomyCar MidSizeCar FullSizeCar < Car .
Each object of class Rental will establish a relationship between a customer and a car,
whose identifiers are kept in attributes customer and car, respectively. In addition to these,
the class Rental is also declared with attributes deposit, pickUpDate, and dueDate to store,
respectively, the amount of money left as a deposit by the customer, the date in which the car
is picked up by the customer, and the date in which the car should be returned to the store.
class Rental | deposit : Nat, dueDate : Nat, pickUpDate : Nat,
customer : Oid, car : Oid .
Giventhesimpleusethatwearegoingtomakeofdates, wecanrepresentthem, forexample,
as natural numbers. Then, we may have a calendar object that keeps the current date and gets
increased by a rewrite rule as follows:2
class Calendar | date : Nat .
rl [new-day] :
< O : Calendar | date : F >
=> < O : Calendar | date : F + 1 > .
Four actions can be identified in our example:
• a customer rents a car,
2We do not worry here about the frequency with which the date gets increased, the possible synchronization
problems in a distributed setting, or with any other issues related to the specification of time. See the papers
[105, 107] on the specification of real-time systems in rewriting logic and Maude for a discussion on these issues.

<!-- manual page 125 -->
6.2. EXAMPLE: A RENT-A-CAR STORE 113
• a customer returns a rented car,
• a customer is suspended for being late in paying her debt or for being late in returning a
rented car, and
• a customer pays (part of) her debt.
The rental of a car by a customer is specified by the car-rental rule below, which involves
the customer renting the car, the car itself (which must be available, i.e., not currently rented),
and a calendar object supplying the current date. The rental can take place if the customer is
not suspended, that is, if her identifier is not in the set of identifiers of suspended customers
of the store, and if the customer has enough cash to make the corresponding deposit. Notice
that a customer could rent a car for less time she really is going to use it on purpose, because
either she does not have enough money for the deposit, or prefers making a smaller deposit.
According to the description of the system, the payment takes place when returning the car,
although there is an extra charge for the days the car was not reserved.
crl [car-rental] :
< U : Customer | cash : M, suspended : false >
< I : Car | available : true, rate : Rt >
< C : Calendar | date : Today >
=> < U : Customer | cash : M - Amnt >
< I : Car | available : false >
< C : Calendar | >
< A : Rental | pickUpDate : Today, dueDate : Today + NumDays,
car : I, deposit : Amnt, customer : U, rate : Rt >
if Amnt := Rt * NumDays /\ M >= Amnt
[nonexec] .
Note that, as already mentioned, those attributes of an object that are not relevant for a
rule do not need to be mentioned. Attributes not appearing in the righthand side of a rule will
maintain their previous values unmodified. Furthermore, since the variables A and NumDays
appear in the righthand side or condition of the rule but not in its lefthand side, this rule has
to be declared as nonexec. Note as well the use of the attributes customer and car in objects
of class Rental, which makes explicit that a rental relationship is between the customer and
the car specified by these attributes.
A customer returning a car late cannot be forced to pay the total amount of money due
at return time. Perhaps she does not have such an amount of money in hand. The return of
a rented car is specified by the rules below. The first rule handles the case in which the car
is returned on time, that is, the current date is smaller than or equal to the due date, and
therefore the deposit is greater than or equal to the amount due.
crl [on-date-car-return] :
< U : Customer | cash : M >
< I : Car | rate : Rt >
< A : Rental | customer : U, car : I, pickUpDate : PDt,
dueDate : DDt, deposit : Dpst >
< C : Calendar | date : Today >
=> < U : Customer | cash : (M + Dpst) - Amnt >
< I : Car | available : true >
< C : Calendar | >
if (Today <= DDt) /\ Amnt := Rt * (Today - PDt)
[nonexec] .
In this case, part of the deposit needs to be reimbursed. We can see that the Rental object
disappears in the righthand side of the rule, that is, it is removed from the set of rentals and
the availability of the car is restored.

<!-- manual page 126 -->
114 CHAPTER 6. OBJECT-ORIENTED MODULES
The second rule deals with the case in which the car is returned late.
crl [late-car-return] :
< U : Customer | debt : M >
< I : Car | rate : Rt >
< A : Rental | customer : U, car : I, pickUpDate : PDt,
dueDate : DDt, deposit : Dpst >
< C : Calendar | date : Today >
=> < U : Customer | debt : (M + Amnt) - Dpst >
< I : Car | available : true >
< C : Calendar | >
if DDt < Today *** it is returned late
/\ Amnt := Rt * (DDt - PDt)
+ ((Rt * (Today - DDt)) * (100 + 20)) quo 100
[nonexec] .
In this case, the customer’s debt is increased by the portion of the amount due not covered by
the deposit.
Debts may be paid at any time, the only condition being that the amount paid is between
zero and the amount of money owed by the customer at that time.
crl [pay-debt] :
< U : Customer | debt : M, cash : N >
=> < U : Customer | debt : M - Amnt, cash : N - Amnt >
if 0 < Amnt /\ Amnt <= N /\ Amnt <= M
[nonexec] .
Customers who are late in returning a rented car or in paying their debts “may” be sus-
pended. The first rule deals with the case in which a customer has a pending debt, and the
second one handles the case in which a customer is late in returning a rented car.
crl [suspend-late-payers] :
< U : Customer | debt : M, suspended : false >
=> < U : Customer | suspended : true >
if M > 0 .
crl [suspend-late-returns] :
< U : Customer | suspended : false >
< I : Car | >
< A : Rental | customer : U, car : I, dueDate : DDt >
< C : Calendar | date : Today >
=> < U : Customer | suspended : true >
< I : Car | >
< A : Rental | >
< C : Calendar | >
if DDt < Today .
Since the system is not terminating, and there are several rules with variables in their right-
hand sides or conditions not present in their lefthand sides and not satisfying the admissibility
conditions discussed in Section 5.3, strategies are necessary for controlling its execution. We
can define many different strategies and use them in many different ways (see Chapter 10 and
Section 17.7); a concrete possibility will be described later in Section 10.1.1.

<!-- manual page 127 -->
6.3. OBJECT-MESSAGE FAIR REWRITING 115
6.3 Object-message fair rewriting
AswesawinSection5.4, inadditiontothestandardrewritingstrategyprovidedbytherewrite
command, Maude provides a command for fair rewriting named frewrite. For regular terms,
the frewrite command (see Section 5.4.2) rewrites a term using a depth-first position-fair
strategy that makes it possible for some rules to be applied that could be “starved” using the
leftmost, outermost rule fair strategy of the rewrite command. For configurations of objects and
messages, the frewrite command provides a rewriting strategy fair for objects and messages.
Theobject-messagefairbehaviorappearswiththecommandfrewrite(andatthemetalevel
with the descent function metaFrewrite—see Section 17.6.3). When the fair traversal attempts
to perform a single rewrite on a term headed by a configuration constructor, the following
happens:
1. Arguments headed by object constructors are collected. It is a runtime error for more
than one object to have the same name.
2. For each object, messages with its name as first argument are collected and placed in a
queue.
3. Any remaining arguments are placed on a remainder list.
4. For each object, and for each message in its queue, an attempt is made to deliver the
message by performing a rewrite with an object-message rule (see below). If all applicable
rulesfail, themessageisplacedontheremainderlist. Ifarulesucceeds, therighthandside
is constructed, reduced, and the result is searched for the object. Any other arguments
in the result configuration go onto the remainder list. If the object cannot be found, any
messages left in its queue go onto the remainder list. Once its message queue is exhausted,
the object itself is placed on the remainder list.
5. A new term is constructed using the configuration constructor on top of the arguments
in the remainder list. This is reduced, and a single rewrite using the non-object-message
rules is attempted.
As we have already explained, and will see in detail in Sections 6.4–6.5, we may have
object-based specifications in regular system modules, and, in fact, object-oriented modules are
transformed into system modules. Thus, the object-message fair strategy is also available on
these object-based system modules. Indeed, it is associated to attributes config, object and
msg, and not to any specific syntax. The syntax provided by the CONFIGURATION module is just
one of the possible alternatives.
The object-message fair rewriting strategy is associated with configuration constructors
which are declared with the config attribute. Configuration constructors must be associative
and commutative, and may optionally have an identity element. The empty syntax constructor
in the CONFIGURATIONmodule above (which has been given the configattribute) is an example
of valid configuration constructor, but such default syntax can easily be changed, e.g., by
renaming the__ operator (see Section 7.3). See an alternative syntax in the data agents
example in Section 7.5.
Configurations only have their special behavior with respect to arguments that are con-
structed using operators that are object or message constructors, that is, they are declared
with the object or message attribute. Such object and message constructors must have at
least one argument. Examples include the Maude object constructor in CONFIGURATION, the
various actor constructors imported into the O-TICKER-FACTORY-TEST module in Section 6.4.2,
all of which have been given the object attribute, and the actor message constructor which
has been given the message/msg attribute.
An operator can have at most one of the three attributes: config, object, and message.
For object constructors, the first argument is considered to be the object’s name. For message
constructors, the first argument is considered to be the message’s target or addressee. There

<!-- manual page 128 -->
116 CHAPTER 6. OBJECT-ORIENTED MODULES
may be multiple configuration, object, and message constructors. A rule is considered to be an
object-message rule if the following requirements hold:
1. its lefthand side has a configuration constructor on top with two arguments A and B;
2. A and B are stable (that is, they cannot change their top symbol under a substitution);
3. A has a message constructor on top;
4. B has an object constructor on top; and
5. the first arguments of A and B are identical.
For example, the rules newReply and timeReply in the O-TICKER-CUSTOMER module (in
Section6.4.2)areobject-messagerules(becauseconfigurationsareassociativeandcommutative,
Aand B can appear in the rule in either order) while the rule labeled req is not, because there
is no message term, only an object, in its lefthand side. This rule will be applied in the rewriting
that happens after all the enabled object-message rules have been applied, as discussed below.
There is no restriction on object names, other than uniqueness. An object may change its
object constructor during the course of a rewrite and delivery of any remaining message will
still be attempted.3 If the configuration constructor changes during the course of a rewrite, the
resulting term is considered alien, and does not participate any further in the object-message
rewriting for the original term. The order in which objects are considered and messages are
delivered is system-dependent, but note that newly created messages are not delivered until
some future visit to the configuration (though all arguments including new messages and alien
configurations could potentially participate in the single non-object-message rewrite attempt).
Message delivery is “just” rather than “fair”: in order for message delivery to be guaranteed, an
object must always be willing to accept the message.4 If multiple object-message rules contain
the same message constructor, they are tried in a round-robin fashion. Non-object-message
rules are also tried in a round-robin fashion for the single non-object-message rewrite attempt.
The counting of object-message rewrites is nonstandard: for the purposes of the rewrite
argumentgiventofrewrite,avisittoaconfigurationthatresultsinoneormorerewritescounts
as a single rewrite, though for other accounting purposes all rewrites are counted. Finally, for
tracing, profiling, and breakpoints only, there is a fake rewrite at the top of the configuration
in the case that object-message rewriting takes place but the single non-object-message rewrite
attempt fails. It is not included in the reported rewrite total, but it is inserted to keep tracing
consistent.
6.4 Object-based programming
As we have explained in the previous sections, object-oriented modules provide a convenient
syntax for the specification of object-oriented systems. However, these modules are just syn-
tactic sugar, and are internally transformed into system modules. Indeed, we can specify
object-based systems directly as system modules, with exactly the same semantics.
6.4.1 Object-based programming using the CONFIGURATION module
Maude supports the modeling of object-based systems through its predefined CONFIGURATION
module (see Section 6.1). The sorts representing the essential concepts of object, message,
and configuration, along with the notation for object syntax serves as a common language
for specifying object-based systems. By using some basic conventions, object-based systems
3Assuming, as it should be the case, that both object constructors have been declared with the object
attribute.
fairness condition [89].
4There are program transformations that internalize conditions on message delivery to ensure a stronger

<!-- manual page 129 -->
6.4. OBJECT-BASED PROGRAMMING 117
specified using system modules have access to the object-message fair rewriting strategy that
is well suited for executing object system configurations.
To specify an object-based system, the user can import the CONFIGURATION module and
then define the particular objects, messages, and rules for interaction that are of interest. The
user is also free to define his/her own notation for configurations and objects, and can still
take advantage of the object-message rewriting strategy, simply by making the appropriate
declarations. See Section 6.4.2 for details on the assumptions.
Note that although Maude does not assume any syntax for messages, the assumption on the
first argument of messages must be respected both in object-oriented modules and in system
modules in order to take advantage of the object-message fair rewriting strategy (see Sec-
tion 6.3). Also, although the user is free to define the syntax of elements of sort Attribute
according to taste, we will follow the standard Maude notation in most of our examples.
The module BANK-ACCOUNT below illustrates the use of the Maude object syntax to define
simple bank account objects, equivalent to the module ACCOUNT presented in Section 6.1.4 using
object-oriented modules. Note that by defining the attribute bal with syntax bal :_ we are
able to write account objects as < A : Account | bal : N >. Note also the gather attribute
to allow writing any expression of the appropriate type without the need of parentheses.
mod BANK-ACCOUNT is
protecting INT .
inc CONFIGURATION .
op Account : -> Cid [ctor] .
op bal :_ : Int -> Attribute [ctor gather (&)] .
ops credit debit : Oid Nat -> Msg [ctor] .
vars A B : Oid .
vars M N N’ : Nat .
rl [credit] :
< A : Account | bal : N >
credit(A, M)
=> < A : Account | bal : N + M > .
crl [debit] :
< A : Account | bal : N >
debit(A, M)
=> < A : Account | bal : N - M >
if N >= M .
endm
The class identifier for bank account objects is Account. Each account object has a single
attribute named bal of sort Int (the account balance). There are two message constructors,
namely credit and debit, each taking an object identifier (the receiver) and a number (the
amount to credit or debit). The rule labeled creditdescribes the processing of a credit message
and the rule labeled debit describes the processing of a debit message. Suppose that constants
A-001, A-002, and A-003 of sort Oid have been declared. Then, the following is an example of
a bank account configuration.
< A-001 : Account | bal : 300 >
< A-002 : Account | bal : 250 >
< A-003 : Account | bal : 1250 >
debit(A-001, 200)
debit(A-002, 400)
debit(A-003, 300)
credit(A-002, 300)

<!-- manual page 130 -->
118 CHAPTER 6. OBJECT-ORIENTED MODULES
Note that the messages debit(A-001, 200) and debit(A-003, 300) can be delivered con-
currently, either before or after the other messages. However, the message debit(A-002, 400)
cannot be delivered until after credit(A-002, 300) has been delivered, due to the balance
condition for the debit rule.
The credit and debit rules are examples of asynchronous message passing rules involving
one object and one message on the lefthand side. In these examples no new objects are created
and no new messages are sent.
In order to combine the debit(A-003, 300) and credit(A-002, 300) messages so that
the delivery of these two messages becomes a single atomic transaction, we could define a new
message constructor from_to_transfer_. The rule for handling a transfer message involves
the joint participation of two bank accounts in the transfer, as well as the transfer message.
This is an example of a synchronous rule.
op from_to_transfer_ : Oid Oid Nat -> Msg [ctor] .
crl [transfer] :
(from A to B transfer M)
< A : Account | bal : N >
< B : Account | bal : N’ >
=> < A : Account | bal : N - M >
< B : Account | bal : N’ + M >
if N >= M .
Now we could replace
debit(A-003, 300)
credit(A-002,300)
by
from A-003 to A-002 transfer 300
in the example configuration. The module BANK-ACCOUNT-TEST declares the object identifiers
introduced above and defines a configuration constant bankConf.
mod BANK-ACCOUNT-TEST is
ex BANK-ACCOUNT .
ops A-001 A-002 A-003 : -> Oid .
op bankConf : -> Configuration .
eq bankConf
= < A-001 : Account | bal : 300 >
debit(A-001, 200)
debit(A-001, 150)
< A-002 : Account | bal : 250 >
debit(A-002, 400)
< A-003 : Account | bal : 1250 >
(from A-003 to A-002 transfer 300) .
endm
From the specification we see that only one of the debit messages for A-001 can be pro-
cessed. Using the default rewriting strategy we find that the message debit(A-001, 150) is
processed first in this strategy.
Maude> rew in BANK-ACCOUNT-TEST : bankConf .
result Configuration:
debit(A-001, 200)
< A-001 : Account | bal : 150 >
< A-002 : Account | bal : 150 >
< A-003 : Account | bal : 950 >

<!-- manual page 131 -->
6.4. OBJECT-BASED PROGRAMMING 119
We use the search command to confirm that it is possible to process the other message
debit(A-001, 200) as well, where the =>! symbol indicates that we are searching for states
reachablefrombankConfthatcannotbefurtherrewritten(seeSection5.4.3andAppendixA.4).
Maude> search bankConf =>! C:Configuration debit(A-001, 150) .
search in BANK-ACCOUNT-TEST : bankConf
=>! C:Configuration debit(A-001, 150) .
Solution 1 (state 8)
states: 9 rewrites: 49 in 0ms cpu (0ms real) (~ rews/sec)
C:Configuration --> < A-001 : Account | bal : 100 >
< A-002 : Account | bal : 150 >
< A-003 : Account | bal : 950 >
No more solutions.
states: 9 rewrites: 49 in 0ms cpu (0ms real) (~ rews/sec)
The BANK-MANAGER module below illustrates asynchronous message passing with object cre-
ation.
mod BANK-MANAGER is
inc BANK-ACCOUNT .
op Manager : -> Cid [ctor] .
op new-account : Oid Oid Nat -> Msg [ctor] .
vars O C : Oid .
var N : Nat .
rl [new] :
< O : Manager | none >
new-account(O, C, N)
=> < O : Manager | none >
< C : Account | bal : N > .
endm
Toopenanewaccount, onesendsamessagetothebankmanagerwiththeaccountnameand
initial balance, for example, new-account(A-000, A-004, 100). Of course, in a real system
more care would be needed to assure unique account identities. To see the bank manager in
action, we define the following module.
mod BANK-MANAGER-TEST is
ex BANK-MANAGER .
ops A-001 A-002 A-003 A-004 : -> Oid .
op mgrConf : -> Configuration .
eq mgrConf
= < A-001 : Account | bal : 300 >
< A-004 : Manager | none >
new-account(A-004, A-002, 250)
new-account(A-004, A-003, 1250) .
endm
Then, we rewrite the configuration mgrConf:
Maude> rew in BANK-MANAGER-TEST : mgrConf .
result Configuration:
< A-001 : Account | bal : 300 >
< A-002 : Account | bal : 250 >
< A-003 : Account | bal : 1250 >
< A-004 : Manager | none >

<!-- manual page 132 -->
120 CHAPTER 6. OBJECT-ORIENTED MODULES
BANK-MANAGER-TEST
BANK-ACCOUNT-TEST BANK-MANAGER
BANK-ACCOUNT
INT
CONFIGURATION
Figure 6.1: Importation graph of bank modules
The relationships between all the modules involved in this example are illustrated in Fig-
ure 6.1, where the different types of arrows correspond to the different modes of importation:
single arrow for including, double arrow for extending, and triple arrow for protecting.
Details on the different forms of importation can be found in Section 7.1.
Note that the example used in this section is very simple. Specifically, all rules presented
use objects for which all their attributes are exhausted. Of course, it is simple if we have one
single attribute as it happens for our bank accounts. Of course, variables of type AttributeSet
can always be used to avoid specifying those attributes not relevant to a rule. But note that
the user has to deal with them, as in any other rule, since rules are not transformed in any way
out of object-oriented modules.
We will explain the automatic transformation that Maude performs for any object-oriented
module in Section 6.5. There we will see how Maude systematically uses variables of sorts Cid
and AttributeSet to model inheritance, and for not requiring the specification of non-relevant
attributes.
6.4.2 Object-based programming without the CONFIGURATION module
The examples above illustrate object-based programming in Maude using the common object
syntax. Alternative object syntax is also possible, by defining an associative and commutative
configuration constructor and suitable object and message syntax. It is of course also possible
not to use the config, object, and message attributes when defining the multiset union
operatorandobjectsandmessageconstructors, butthiswillpreventtakingadvantageofobject-
message fair rewriting (see Section 6.3).
In this section, we are going to illustrate the importance of following the assumptions so
that the object-message fair rewriting strategy works properly. But let us make it clear once
more, despite the assumptions made in previous sections, the user is free to define any object
or message syntax that is convenient. However, the strategy needs to clearly identify what is
a configuration, what is an object and a message, and what are the identifiers of object and
addressees of messages. Recall that, for uniformity in identifying objects and message receivers,

<!-- manual page 133 -->
6.4. OBJECT-BASED PROGRAMMING 121
the adopted convention is that configurations, objects, and messages must be identified, and the
first argument of an object or message constructor must be an object’s name. This facilitates
defining object system rewriting strategies independently of the particular choice of syntax
and is essential for using Maude’s object-message fair rewriting strategy. Even though we will
come back to this below, let us be specific. If a syntax different to the one introduced by the
CONFIGURATION module is used, the following conditions must be followed: object constructors
must be given the attribute object, message constructors must be given the attribute message,
the constructor of object and message configurations must be given the config attribute, the
first argument of object constructors must be the identifier of the object, and the first argument
of a message must be the identifier of its addressee object.
As an example (not using for the moment these attributes, to illustrate different forms of
rewriting with objects), we model a ticker, the classic example of an actor [1, 112]. First, we
specify the configurations, objects, and messages of the actor world in the module ACTOR-CONF.
Actor configurations (of sort AConf) are multisets of actors (of sort Actor) and messages (of
sort Msg). Messages are constructed uniformly from an actor identifier and a message body.
Thus we introduce sorts Aid (actor identifier) and MsgBody, and a message constructor _<|_.
mod ACTOR-CONF is
sorts Actor Msg AConf .
subsorts Actor Msg < AConf .
op none : -> AConf [ctor] .
op __ : AConf AConf -> AConf [ctor assoc comm id: none] .
*** actor messages
sorts Aid MsgBody .
op _<|_ : Aid MsgBody -> Msg [ctor] .
endm
A ticker maintains a counter that it updates in response to a tick message. The term
Ticker(T:Aid, N:Nat) represents an actor with identifier T:Aid and counter value N:Nat.
The ticker sends the current value of its counter in response to a timeReq message.
mod TICKER is
including ACTOR-CONF .
protecting NAT .
op Ticker : Aid Nat -> Actor [ctor] .
op tick : -> MsgBody [ctor] .
op timeReq : Aid -> MsgBody [ctor] .
op timeReply : Nat -> MsgBody [ctor] .
vars T C : Aid .
var N : Nat .
rl Ticker(T, N) (T <| tick)
=> Ticker(T, s N) (T <| tick) .
rl Ticker(T, N) (T <| timeReq(C))
=> Ticker(T, N) (C <| timeReply(N)) .
endm
To test the ticker we define actor identifiers for the ticker, myticker, a customer, me, and
an initial configuration with one ticker, one tick message, and a timeReq message from me.
mod TICKER-TEST is
extending TICKER .
ops myticker me : -> Aid [ctor] .
op tConf : -> AConf .
eq tConf

<!-- manual page 134 -->
122 CHAPTER 6. OBJECT-ORIENTED MODULES
= Ticker(myticker, 0)
(myticker <| tick)
(myticker <| timeReq(me)) .
endm
If we ask Maude to rewrite the configuration tConf without placing an upper bound on
the number of rewrites, Maude will go on forever. This is because there will always be a tick
message in the configuration, and the ticker can always process this message. Thus we rewrite
with an upper bound of, say, 10 rewrites.
Maude> rew [10] tConf .
rewrite [10] in TICKER-TEST : tConf myticker <| timeReq(me) .
result AConf:
(myticker <| tick)
(me <| timeReply(1))
Ticker(myticker, 9)
We see that the timeReq message was processed after just one tick was processed.
An interesting property of this configuration is that the reply to the timeReq message can
contain an arbitrarily large natural number, since any number of ticks could be processed
before the timeReq is processed. For particular numbers this can be checked using the search
command.
Maude> search [1] tConf =>+ tc:AConf me <| timeReply(100) .
Solution 1 (state 5152)
states: 5153 rews: 5153 in 0ms cpu (285ms real)(~ rews/sec)
tc:AConf --> (myticker <| tick) Ticker(myticker, 100)
Notice that we used the search relation =>+ (one or more steps) rather than =>! (terminating
rewrites) since there are no terminal configurations starting from tConf. Moreover, we have
searched only for the first ([1]) solution.
Next, the ticker example defines a ticker factory that creates tickers, and a ticker-customer.
The ticker factory accepts requests for new tickers newReq(c) where c is the customer’s name.
When such a request is received, a ticker is created and its name is sent to the requesting
customer (newReply(o(a, i))). To make sure that each ticker has a fresh (unused) name,
the ticker factory keeps a counter. It generates ticker names of the form o(a, i), where a is
the factory name and i is the counter value. The counter is incremented each time a ticker is
created. This is just one possible method for assuring unique names for dynamically created
objects. Ifobjectsareonlycreatedbyfactoriesthatusetheabovemethodforgeneratingnames,
then starting from a configuration of objects with unique names (not of the form o(a, i)) the
unique name property will be preserved.
mod TICKER-FACTORY is
inc TICKER .
op TickerFactory : Aid Nat -> Actor [ctor] .
ops newReq newReply : Aid -> MsgBody [ctor] .
op o : Aid Nat -> Aid [ctor] .
vars A C : Aid .
vars I J : Nat .
rl [newReq] :
TickerFactory(A, I) (A <| newReq(C))
=> TickerFactory(A, s I) (C <| newReply(o(A, I)))

<!-- manual page 135 -->
6.4. OBJECT-BASED PROGRAMMING 123
Ticker(o(A, I), 0) (o(A, I) <| tick) .
endm
A ticker customer knows the name of a ticker factory. It asks for a ticker, waits for a reply,
asks the ticker for the time, waits for a reply, increments its reply counter (used just for the
user to monitor customer service) and repeats this process.
mod TICKER-CUSTOMER is
inc TICKER-FACTORY .
ops Cust Cust1 Cust2 : Aid Aid Nat -> Actor [ctor] .
vars C TF T : Aid .
vars N M : Nat .
rl [req] :
Cust(C, TF, N)
=> Cust1(C, TF, N) (TF <| newReq(C)) .
rl [newReply] :
Cust1(C, TF, N) (C <| newReply(T))
=> Cust2(C, TF, N) (T <| timeReq(C)) .
rl [timeReply] :
Cust2(C, TF, N) (C <| timeReply(M))
=> Cust(C, TF, s N) .
endm
Nowwedefineatestconfigurationwithatickerfactoryandtwocustomers. Theimportation
graph of all the modules involved at this point is shown in Figure 6.2.
mod TICKER-FACTORY-TEST is
ex TICKER-CUSTOMER .
ops tf c1 c2 : -> Aid [ctor] .
ops ic1 ic2 : -> AConf .
eq ic1 = TickerFactory(tf, 0) Cust(c1, tf, 0) .
eq ic2 = ic1 Cust(c2, tf, 0) .
endm
Rewriting this configuration using the rewrite command with a bound of 40 results in one
ticker being created, and ticking away, while customer c2 is not given an opportunity to execute
at all.
Maude> rew [40] ic2 .
rewrite [40] in TICKER-FACTORY-TEST : ic2 .
rewrites: 42 in 0ms cpu (0ms real) (~ rewrites/second)
result AConf:
(o(tf, 0) <| tick)
Ticker(o(tf, 0), 35) TickerFactory(tf, 1)
Cust(c1, tf, 1) Cust(c2, tf, 0)
In contrast, rewriting using the frewrite strategy with the same bound of 40, several tickers
are created, however only the first one gets tick messages delivered.
Maude> frew [40] ic2 .
frewrite [40] in TICKER-FACTORY-TEST : ic2 .
rewrites: 42 in 0ms cpu (1ms real) (~ rewrites/second)
result (sort not calculated):

<!-- manual page 136 -->
124 CHAPTER 6. OBJECT-ORIENTED MODULES
TICKER-FACTORY-TEST
TICKER-CUSTOMER
TICKER-TEST TICKER-FACTORY
TICKER
NAT
ACTOR-CONF
Figure 6.2: Importation graph of ticker modules
(o(tf, 0) <| tick) (o(tf, 1) <| tick)
(o(tf, 2) <| tick) (o(tf, 3) <| tick)
(o(tf, 4) <| tick) (o(tf, 5) <| tick)
(o(tf, 6) <| tick)
(o(tf, 6) <| timeReq(c1))
Ticker(o(tf, 0), 6) Ticker(o(tf, 1), 0)
Ticker(o(tf, 2), 0) Ticker(o(tf, 3), 0)
Ticker(o(tf, 4), 0) Ticker(o(tf, 5), 0)
Ticker(o(tf, 6), 0)
TickerFactory(tf, 7)
((tf <| newReq(c2))
Cust1(c2, tf, 3)) Cust2(c1, tf, 3)
The number of rewrites reported by Maude includes both equational and rule rewrites.
In the examples above there were two equational rewrites (the two equations defining the
initial configuration ic2 and its subconfiguration ic1) and 40 rule rewrites. If you execute the
command
Maude> set profile on .
(see Section 21.1.5) before rewriting and then execute
Maude> show profile .
you will discover that executing the rewrite command the rule delivering the tick message is
used 35 times and the other rules are each used once, while executing the frewrite command
the tick rule is executed only 6 times and each of the other rules are executed between 6 and
8 times.

<!-- manual page 137 -->
6.4. OBJECT-BASED PROGRAMMING 125
Turning profiling on substantially reduces performance, so you will want to turn it off
Maude> set profile off .
when you have found out what you want to know.
Note that frewrite uses a fair rewriting strategy, but since it does not know about objects,
messages, and configurations, it can only follow a position-fair strategy. As we explain in
Section 6.3, in order to enable the object-message fair rewriting strategy we need only do three
things:
• give to the constructor of object and message configurations the config attribute,
• give to the message constructor the message attribute, and
• give to each object constructor the object attribute.
To maintain the separate rewriting semantics we also modify the name of each module by
putting an O at the front (except for ACTOR-CONF which we rename ACTOR-O-CONF). Thus we
modify the configuration, actor, and message constructor declarations as follows.
mod ACTOR-O-CONF is
...
op __ : AConf AConf -> AConf [ctor config assoc comm id: none] .
op _<|_ : Aid MsgBody -> Msg [ctor message] .
...
endm
mod O-TICKER is
...
op Ticker : Aid Nat -> Actor [ctor object] .
...
endm
mod O-TICKER-FACTORY is
...
op TickerFactory : Aid Nat -> Actor [ctor object] .
...
endm
mod O-TICKER-CUSTOMER is
...
ops Cust Cust1 Cust2 : Aid Aid Nat -> Actor [ctor object] .
...
endm
Now the frewrite command will use object-message fair rewriting, as explained in detail
in Section 6.3. The counting of object-message rewrites has two aspects: for the purposes
of the rewrite argument given to frewrite, a visit to a configuration that results in one or
more rewrites counts as a single rewrite; though for other accounting purposes all rewrites are
counted. For example, with an upper bound of 40 as above, thirteen tickers are created. To
simplify the output we show the results for rewriting with a bound of 20.
Maude> frew [20] ic2 .
frewrite [20] in O-TICKER-FACTORY-TEST : ic2 .
rewrites: 76 in 0ms cpu (1ms real) (~ rewrites/second)
result (sort not calculated):
(o(tf, 0) <| tick) (o(tf, 1) <| tick)
(o(tf, 2) <| tick) (o(tf, 3) <| tick)
(o(tf, 4) <| tick) (o(tf, 5) <| tick)

<!-- manual page 138 -->
126 CHAPTER 6. OBJECT-ORIENTED MODULES
Ticker(o(tf, 0), 11) Ticker(o(tf, 1), 11)
Ticker(o(tf, 2), 7) Ticker(o(tf, 3), 7)
Ticker(o(tf, 4), 3) Ticker(o(tf, 5), 3)
TickerFactory(tf, 6)
((tf <| newReq(c1)) Cust1(c1, tf, 3))
(tf <| newReq(c2)) Cust1(c2, tf, 3)
Notice that each ticker gets a chance to tick (tickers created later will show less time passed),
and each customer is treated fairly. In fact using profiling we find that the tick rule is used
42 times (which is the total of the counts for the six tickers created), while the other rules are
used 6-8 times and there are two equational rewrites as before.
Suppose that we try to violate the unique name condition, for example by adding a copy of
customer c1 to the test configuration. When Maude discovers this (it may take a few rewrites),
a warning is issued.
Maude> frew [4] ic2 Cust(c1, tf, 0) .
Warning: saw duplicate object: Cust1(c1, tf, 0)
frewrite [4] in O-TICKER-FACTORY-TEST : ic2 Cust(c1, tf, 0) .
rewrites: 8 in 0ms cpu (0ms real) (~ rewrites/second)
result AConf:
(c1 <| newReply(o(tf, 0))) (c1 <| newReply(o(tf, 1)))
(c2 <| newReply(o(tf, 2)))
(o(tf, 0) <| tick) (o(tf, 1) <| tick) (o(tf, 2) <| tick)
Ticker(o(tf, 0), 0) Ticker(o(tf, 1), 0) Ticker(o(tf, 2), 0)
TickerFactory(tf, 3)
Cust1(c1, tf, 0) Cust1(c1, tf, 0) Cust1(c2, tf, 0)
6.5 From object-oriented modules to system modules
The best way to understand classes and class inheritance in Maude is by making explicit the
full structure of an object-oriented module, which is left somewhat implicit by the syntactic
conventions adopted for them. Indeed, although object-oriented modules provide convenient
syntax for programming object-oriented systems, their semantics can be reduced to that of
system modules. We can regard the special syntax reserved for object-oriented modules as
syntactic sugar. In fact, each object-oriented module can be translated into a corresponding
system module whose semantics is by definition that of the original object-oriented module.
In the translation process, the most basic structure shared by all object-oriented modules
is made explicit by the CONFIGURATION functional module (see Section 6.1. The translation
of a given object-oriented module extends this structure with the classes, messages, and rules
introduced by the module, following the conventions explained in Section 6.4.
The transformed version of a module can be obtained using the show desugared command.
For example, the following command shows the system module resulting from the translation
of the ACCOUNT module introduced in Section 6.1. Note that a subsort Account of Cid is
introduced. The purpose of this subsort is to range over the class identifiers of the subclasses
of Account. For the moment, no such subclasses have been introduced; therefore, at present
the only constant of sort Account is the class identifier Account.
5
Maude> show desugared .
mod ACCOUNT is
5The ACCOUNT module imports modules BOOL, CONFIGURATION, QID, and NAT in different modes. See Section 7.1
for details on the different forms of importation.

<!-- manual page 139 -->
6.5. FROM OBJECT-ORIENTED MODULES TO SYSTEM MODULES 127
including BOOL .
including CONFIGURATION .
protecting QID .
protecting INT .
sort Account .
subsort Qid < Oid .
subsort Account < Cid .
op credit : Oid Int -> Msg [ctor msg] .
op debit : Oid Int -> Msg [ctor msg] .
op from_to_transfer_ : Oid Oid Int -> Msg [ctor msg prec 41 gather (& & E)] .
op Account : -> Account [ctor] .
op bal‘:_ : Int -> Attribute [ctor prec 15 gather (&)] .
var A : Oid .
var B : Oid .
var N : Int .
var M : Nat .
var N’ : Int .
rl [credit] :
credit(A, M)
< A : V:Account | Atts:AttributeSet, bal : N >
=> < A : V:Account | Atts:AttributeSet, bal : (M + N) > .
crl [debit] :
debit(A, M)
< A : V:Account | Atts:AttributeSet, bal : N >
=> < A : V:Account | Atts:AttributeSet, bal : (N - M) >
if N >= M = true .
crl [transfer] :
< A : V:Account | Atts:AttributeSet, bal : N >
< B : V2:Account | Atts2:AttributeSet, bal : N’ >
from A to B transfer M
=> < A : V:Account | Atts:AttributeSet, bal : (N - M) >
< B : V2:Account | Atts2:AttributeSet, bal : (M + N’) >
if N >= M = true .
endm
We can describe the desired transformation from an object-oriented module to a system
module as follows:6
• The module CONFIGURATION from Section 6.1 is imported.
• For each class declaration of the form
class C | a1 : S1, ..., an : Sn .
the following is introduced: a subsort C of sort Cid, a constant C of sort C, and declara-
tions of operations
op ai :_ : Si -> Attribute [ctor gather (&)] .
for each attribute ai.
7
• For each subclass relation C < D a subsort declaration
subsort C < D.
6Thisprocesssimplifiesthetransformationofobject-orientedmodulesintosystemmodulesthatwasoriginally
presented in [93].
7Ad hoc overloading of attributes is allowed—internally they are considered to be completely different at-
tributes that just happen to share a name. There can be subsort polymorphic overloading of attributes from
different classes.

<!-- manual page 140 -->
128 CHAPTER 6. OBJECT-ORIENTED MODULES
is introduced.
• Thesystemmodulesresultingfromthetransformationhavethespecialfeaturessupported
for object-based programming explained in Section 6.4. Specifically, the msg attribute is
added to message declarations starting with the msg keyword.
• Thestatementsaremodifiedtomakethemapplicabletoallobjectsofthegivenclassesand
of their subclasses, that is, not only to objects whose class identifiers are those explicitly
given. Therulesarethen“inherited” byallobjectsintheirsubclassesbyreplacingtheclass
identifiers in the objects in the rules by variables of the corresponding class sort. Variables
ofsortAttributeSetarealsointroduced, torangeovertheadditionalattributesthatmay
appear in objects of a subclass. Furthermore, statements are modified to give the user
the possibility of not mentioning in a given statement those attributes of objects that are
not relevant for that rule. Specifically, statements (equations, rules, and memberships)
that satisfy the transformation requirements (see Section 6.1) are transformed as follows:
– If the class argument of an existing object E is a class constant C, it is replaced by
a fresh variable V : C in all occurrences of E.
– If the attribute arguments of the occurrences of an existing object E lack a variable
of type AttributeSet, a fresh variable Atts:AttributeSet is added to all such
attribute arguments.
– All occurrences of the constant none are removed from the attribute argument of all
occurrences of existing objects.
– If the attribute argument of the pattern occurrence of some existing object E has no
term headed by attribute operator a :_ while some other occurrence of E has such
a term in its attribute argument, a term a : A:[K] is added where A:[K] is a
fresh variable of the appropriate kind.
– If the attribute argument of a subject or condition-subject occurrence of some ex-
isting object E has no term headed by attribute operator a :_ while the completed
pattern occurrence of object E does have such a term a : u(which may have been
generated by the previous step) then a copy of a : u is added to its attribute
argument.
Note that the transformation on candidate statements is idempotent—a fully completed
statement meets all the requirements but the transformation is the identity.
If a statement is a candidate, and meets all the requirements, no advisories are printed
and any transformation is done silently. Details on the transformation of the object-oriented
module may be obtained using the set verbose on command (see Appendix A.15). It will
cause the transformation process to print a summary for each candidate statement meeting the
requirements.
The rewrite rules given in the original ACCOUNT module are interpreted here—according to
the conventions already explained—in a form that can be inherited by subclasses of Account
that could be defined later. Thus, SavingAccount inherits the rewrite rules for crediting
and debiting accounts, and for transferring funds between accounts that had been defined for
Account.
Let us illustrate the treatment of class inheritance with the system module resulting from
the transformation of the module SAVING-ACCOUNT introduced previously.
Maude> show desugared .
mod SAVING-ACCOUNT is
including BOOL .
including CONFIGURATION .
including ACCOUNT .
protecting FLOAT .
sort SavingAccount .

<!-- manual page 141 -->
6.5. FROM OBJECT-ORIENTED MODULES TO SYSTEM MODULES 129
subsort SavingAccount < Cid .
subsort SavingAccount < Account .
op SavingAccount : -> SavingAccount [ctor] .
op rate‘:_ : Float -> Attribute [ctor prec 15 gather (&)] .
endm
Note that by translating a rule like credit above
rl [credit] :
credit(A, M)
< A : Account | bal : N >
=> < A : Account | bal : (N + M) > .
into its corresponding transformed form
rl [credit] :
credit(A, M)
< A : V:Account | Atts:AttributeSet, bal : N >
=> < A : V:Account | Atts:AttributeSet, bal : (M + N) > .
it is guaranteed that the rule will be applicable to objects of class Account as well as of any of
its subclasses.
Note also that a rule like change-age (discussed in Section 6.1.4)
rl [change-age] :
< O : Person | >
to O : new age A
=> < O : Person | age : A > .
is translated into a rule like
rl [change-age] : (to O : new age A)
< O : V:Person | Atts:AttributeSet, age : A2:[FindResult] >
=> < O : V:Person | Atts:AttributeSet, age : A > .
With this translation we allow the rule to be applied to objects in subclasses of Person.
See [37] for a detailed explanation of the transformation of object-oriented modules into
system modules and how their semantics is by definition that of the original object-oriented
module.

<!-- manual page 142 -->
130 CHAPTER 6. OBJECT-ORIENTED MODULES

