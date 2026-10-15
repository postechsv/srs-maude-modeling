[Maude Types Extraction]
Extract from the Maude model the common data types needed to write initial states and search commands without reading the model's rules and equations.

# Input
runs/model.maude

# Output
runs/maude_types.yaml — in the following format.

module: <name of the module in model.maude>
imports: [<modules the model imports, e.g. INT, STRING, QID>]
sorts: [<sort names>]
subsorts: ['<subsort declaration, e.g. subsort Qid < Oid>']
constructors:
  - signature: '<op declaration with the ctor attribute, e.g. op date : String -> Date [ctor]>'
    meaning: <how values of this sort are written>
classes:
  - name: <class name>
    identity: '<how an object of this class is identified, with an example>'
    attributes:
      - name: <attribute name>
        sort: <sort>
        holds: <only for an attribute that holds other objects or their identities: which class, and whether the objects or their identities>
messages:
  - signature: '<msg declaration>'
    arguments: [<name and role of each argument, in order>]
    meaning: <what the message requests>

# Conditions
1. Include only the module name, imported modules, sorts, constructors, classes, and messages.
2. Write subsorts only if model.maude declares subsorts. Otherwise, omit subsorts.
3. Write holds only for an attribute that holds other objects or their identities.
4. In arguments, give the name and role of every argument of the message, in the order of the declaration.
5. In meaning, identity, holds, and arguments, describe only how things are written and what is requested. Do not describe conditions, calculations, or how any rule behaves.
6. Do not include rules, equations, or any operator that is not a constructor.
