# Widely used intertwined integration types
> This are the commonly used types among different platforms for integrated applications and may be added as overloading for methods specialized on dealing with each data type possibly received from a transmission

```lua
/* based upon */ :u8 / /*C#*/(byte) / /*JS*/:Number

-- abstract type     flag          /\/     .NET     SQL             TS
/* integer  */ = | 11111**1 | --   249      int     int            Number*
/* decimal  */ = | 1111**1* | --   242     float    numeric(10,2)  Number
/* timing   */ = | 111**1** | --   228    DateTime  DateTime*      Date*
/* identity */ = | 11**1*** | --   200      Guid    identifier     String*
/* textual  */ = | 1**1**** | --   144     String   varchar*       String
```
- integer **Number\*** should have it's own Value-Type ensuring non decimal parts
- timing **Date\*** should have it's own Valye-Type system based on the default date value (for durations)
- identity **UUID\*** should have it's own Valye-Type (as Guid)

> Logical types are dealt with an integer value wich 1 is true and zero is false anything else should be dimmed as
corrupted data.
---
## Const well known value rules
> The data about the transmitted should follow a rule contract before sending. This contract should be centralized in a particular provider, applications may store this locally but it must be provided externally.

```lua
/* based upon */ :int32 / /*C#*/<int> / /*JS*/:Number*

-- TG        Name          length or id u16    binary flag u16         /\/   meaning
/* #N~ */ NonNegative  = | ******** ******** ******** *******1 | --     1    Number not negative
/* #N+ */ MoreThanZero = | ******** ******** ******** ******1* | --     2    Number more than zero
/* #AA */ NotEmpty     = | ******** ******** ******** *****1** | --     4    Text not empty nor only spaces
/* #RQ */ Required     = | ******** ******** ******** 1******* | --    128   A not default value must be set
/* #NN */ PoliName     = | ******** ******** *******1 ****1*** | --    264   Requires multi part name
/* #NS */ NoSpaces     = | ******** ******** ******** ****1*** | --     8    Continuous text without spaces
/* #AB */ LettersOnly  = | ******** ******** ****111* ***1**** | --   3600   Text only letters
/* #Nº */ NumberOnly   = | ******** ******** *****11* ***1**** | --   1552   Text only ordinal numbers
/* #AP */ Alphanumeric = | ******** ******** ******1* ***1**** | --    528   Alphanumeric text
/* #T- */ FutureTime   = | ******** ******** ***1**** **1***** | --   4128   Date/Time of the future
/* #T+ */ PastTime     = | ******** ******** ******** **1***** | --     32   Date/Time from past
/* #!= */ Unique       = | ******** ******** 111***** *1****** | --  57408   NotNull unique index
/* #PK */ PrimaryKey   = | ******** ******** *11***** *1****** | --  24640   Primary unique key
/* #FK */ ForeignKey   = | ******** ******** **1***** *1****** | --   8256   Foreign relational key

/* #LN */ LineLength   = | XXXXXXXX XXXXXXXX ******** ******** | --   ????   Max length indicator (x)
/* #CS */ Customized   = | XXXXXXXX XXXXXXXX ******** ******** | --     0    CUSTOM RULES         (y)
```

- #CS This is an non-transmittable rule first two bytes indicate their unique id at context THIS RULE IS NOT COMBINABLE its value must be exact in order to assume its actually it
- #LN The first two bytes are reserved to dictate the actual max length of the value. This also might be used to demonstrate the repeating length of types other than text. If the first two bytes of the rule is the max value of u16 then it means that the length must be provided as a header to this field. This rule is combinable, all the other rules
combined to this one must apply upon all elements

# The value contract system

Each contract represents an entity instances clas and should be transmitted as

```lua
-- the contract doens't have a name or description by itself, it might be provided by the context
-- b-len type       meaning 
    1    :u8      -- THE SIGNAL
    4    :i32     -- The rule-set
    16?  :u8.guid -- Maybe the type id or ascii characters
```

### (0-1) The Signal
Either 0, 1 or anything else. 
- Zero means that the contract ended and more irrelevant trimming data might be added
- One means that the type system is complex and the type GUID will be added
- Else cast to ***integration types***

### (2-4) The RuleSet
Addresses one or more set of rules associated with the given value
* seealso ***Const well known value rules***

### (5-21) The TypeId?
May exists and may not based on the signal's value. It might be either random bytes for a unique identifier or ascii characters. Which one makes sense first

# Contract based value transmission

There are two possibilities, either the listener knows what type should it receive or its a generic one. For the known one, a instance sequence is provided abstract bytes in sequence, the actual meaning of those will be interpreted by the listener with the entity contract in hands. If its a generic one than the contract ID should be the first 16 bytes of the message's content