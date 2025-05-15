# Moldo (Mold Objects ) // molde de objetos
> This is a convention file to deal with objects model design through a contract knowledge system. The objective of this system is enabling multi platform applications to communicate with each other using the same language. The type system here is not entirely dynamic but there might be new extra columns to deal with without recompiling the code

## Widely used intertwined integration types
> This are the commonly used types among different platforms for integrated applications and may be added as overloading for methods specialized on dealing with each data type possibly received from a transmission

```lua
/* based upon */ :u8 / /*C#*/(byte) / /*JS*/:Number

-- abstract type     value         /\/     .NET     SQL             TS
/* decimal  */ = | 11110010 | --   242     float    numeric(10,2)  Number
/* tempora  */ = | 11100100 | --   228    DateTime  DateTime*      Date*
/* identity */ = | 11001000 | --   200      Guid    identifier     String*
/* textual  */ = | 10010000 | --   144     String   varchar*       String
```
- decimal **Number\*** should have it's own Value-Type ensuring non decimal parts
- temporal **Date\*** should have it's own Valye-Type system based on the default date value (for durations)
- identity **UUID\*** should have it's own Valye-Type (as Guid)

> Logical types are dealt with an integer value wich 1 is true and zero is false anything else should be dimmed as
corrupted data.
---
## Const well known value rules
> The data about the transmitted should follow a rule contract before sending. This contract should be centralized in a particular provider, applications may store this locally but it must be provided externally.

```lua
/* based upon */ :int32 / /*C#*/<int> / /*JS*/:Number*

-- TG        Name          length or id u16  01234567 89ABCDEF      meaning
/* #N~ */ NonNegative  = | ******** ******** ******** *******1 | --  Number not negative
/* #N+ */ NonZeroed    = | ******** ******** ******** ******1* | --  Value not zero or default
/* #AA */ NotEmpty     = | ******** ******** ******** *****1** | --  trimmed text not empty or values not default
-- --------------------------------spaces management------------------------------------------------
/* #NN */ PoliName     = | ******** ******** *******1 ****1*** | --  Requires multi part name
/* #NS */ NoSpaces     = | ******** ******** ******** ****1*** | --  Continuous text without spaces
-- ------------------------------characters management----------------------------------------------
/* #AB */ LettersOnly  = | ******** ******** ****111* ***1**** | --  Text only letters
/* #Nº */ NumberOnly   = | ******** ******** *****11* ***1**** | --  Text only ordinal numbers
/* #AP */ Alphanumeric = | ******** ******** ******1* ***1**** | --  Alphanumeric text
-- ----------------------------------time control---------------------------------------------------
/* #T- */ FutureTime   = | ******** ******** ***1**** **1***** | --  Date/Time of the future
/* #T+ */ PastTime     = | ******** ******** ******** **1***** | --  Date/Time from past
-- -----------------------------------relational----------------------------------------------------
/* #!= */ Unique       = | ******** ******** 111***** *1****** | --  Values cannot repeat within collection
/* #PK */ PrimaryKey   = | ******** ******** *11***** *1****** | --  Values cannot repeat within repository
/* #FK */ ForeignKey   = | ******** ******** **1***** *1****** | --  External key inforced

-- ----------------------------------value ranges---------------------------------------------------
/* #>> */ MaxValue     = | XXXXXXXX XXXXXXXX *******1 1******* | --  Max Value indicator
/* #<< */ MinValue     = | XXXXXXXX XXXXXXXX ******** 1******* | --  Min Value indicator
/* #<> */ LineLength   = | XXXXXXXX XXXXXXXX ******** ******** | --  Max length indicator (x)
/* #?? */ Customized   = | XXXXXXXX XXXXXXXX 00000000 00000000 | --  CUSTOM RULES         (y)
```

---

### The first region
1. Upon Text 
> _NotEmpty_ requires a text to be null when there are no characters. If "NonZeroed" is also specified than the text cannot be null at all.
2. Upon Numbers
> They do as said, but _NotEmpty_ acts as a **not null** specifier
3. Upon Dates
> _NonNegative_ indicates that its actually a duration type (subtract default and you'll have a period) _NotEmpty_ becomes a **not null** specifier, _NonZeroed_ informs that it cannot be the **default** value

---

### Spaces management
1. Upon Text
> Expected behavior, they conflict between each other and the greater one should be check first
2. Upon Numbersflo
> _PoliName_ demonstrates a decimal-part containing number (non-integer) and _NoSpaces_ is the opposite. (none remains decimal number)
3. Upon Dates
> _PoliName_ demonstrates a DateTime value and _NoSpaces_ a DateOnly kind of value. (none remains DateTime)

---

### Characters management
1. Upon Text
> This is the only section where this region is relevant. They are not combinable. They do allow spaces _NoSpaces_ is not specified. Alphanumeric allows ONLY letters and numbers. None of em allow the whole ansi character set

---

### Time control
1. Upon Dates
> This is the only section where this region is relevant. They are not combinable. _FutureTime_ allows only value after the current time of the server and _PastTime_ allows only values before it 

---

### Relational
1. \* (Any)
> These are special cases for collections, which are either duplicates within the repository or within a collection itself

---

### Value Ranges
1. \* (Any)
> _MaxValue_, _MinValue_ uses the left side of the **i32** to declare a boundary values for an number if applied on text will be interpreted as max length. _LineLength_ serves the same purpose but works for both text and numbers, in case of text represents string length and other types is instance count. **Customized** carries a part of a on the left side of the **i32** that identifies a custom rule-set

int32 value rules bit break down flag indicators:
```
byte 1
    b00       NonNegative 
    b01       NonZeroed   
    b02       NotEmpty
    b03       group S present indicator
    -
    b04       group T present indicator
    b05       group U present indicator
    b06       group V present indicator
    b07       group W/L switcher indicator (ON/ValueRange OFF/MaxLength)

byte 2
    b08       group S and W switcher (ON/PoliName OFF/NoSpaces) (ON/MaxValue OFF/MinValue)
    b09       group T discriminator T'
    b10       group T discriminator T''
    b11       group T discriminator T''' (sum T(s) 1-Alphanumeric 2-NumerOnly 3-LettersOnly)
    -
    b12       group U switcher (ON/FutureTime OFF/PastTime)
    b13       group V indicator V'   Unique
    b14       group V indicator V''  PrimaryKey
    b15       group V indicator V''' ForeignKey
    
[   bRR 16 more bits ] used to indicate a numeric value to indicate either 

- Max/Min Value (when w group is ON)
- Maximum array Length (when w group is OFF)
- Custom rule-set Id when all other groups, switches, indicadators and discriminators are set to 0

```

Example interpretation of { 10010000, 00000000_00010000_00000011_00011000  }
```lua

10010000 == TEXT

right region =
    b03     /* group S present indicator */
    b04     /* group T present indicator */

    b08 ON  /* group S and W switcher (ON/PoliName OFF/NoSpaces) (ON/MaxValue OFF/MinValue) */
    b09 ON  /* group T discriminator T' */

-- (sum T(s) 1-Alphanumeric 2-NumerOnly 3-LettersOnly)

    bRR = 00000000-00010000 / 16
```
which translates to the following rule: A text, with more than one word, only numbers & letters and a maximum 
length of 16

---

# The value contract system

Each contract represents an entity instances clas and should be transmitted as

```lua
-- the contract doens't have a name or description by itself, it might be provided by the context
-- b-len type       meaning 
    1    :u8      -- The Type Signal
    4    :i32     -- The rule-set
    16?  :u8.guid -- Maybe the type id or ascii characters
```

### (0-1) The Type Signal
Either 0, 1 or anything else. 
- Zero means that the contract ended and more irrelevant trimming data might be added
- One means that the type system is complex and the type GUID will be added
- Else cast to ***integration types***

### (2-4) The RuleSet
Addresses one or more set of rules associated with the given value
* seealso ***Const well known value rules***

### (5-21) The TypeId?
May exists and may not based on the signal's value. It might be either random bytes for a unique identifier or ascii characters. Which one makes sense first

---

## Contract based value transmission

There are two possibilities, either the listener knows what type should it receive or its a generic one. For the known one, a instance sequence is provided abstract bytes in sequence, the actual meaning of those will be interpreted by the listener with the entity contract in hands. If its a generic one than the contract ID should be the first 16 bytes of the message's content