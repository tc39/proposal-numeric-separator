# Numeric Separators

This is a [stage 1 proposal](https://tc39.github.io/process-document/) to introduce a separator in numeric literals in [ECMAScript](https://github.com/tc39/ecma262/).

## Motivation

This feature enables developers to make their numeric literals more readable by creating a visual separation between groups of digits.

For example:

```js
var thousands = 10_000; // Instead of 10000.
var credit_card_number = 1234_5678_9012_3456; // Instead of 123456789012345.
var social_security_number = 999_99_9999; // Instead of 999999999.
var pi = 3.14_15; // Instead of 3.1415
var bytes = 0b11010010_01101001_10010100_10010010; // Instead of 0b11010010011010011001010010010010.
var 0xCAFE_F00D; // Instead of 0XCAFEF00D.
```

## Strawman

### Semantics

This feature is designed to have no impact on the interpretation semantics of numeric literals: `_` are to be ignored by interpreters and should have no effect. They are meant **exclusively** as a visual clue to aid development and have no runtime semantics.

### Syntax

We want to optimize to cover the most common use cases and discourage patterns that would be frowned upon in style guides later on.

With that in mind, here is what we think is a good balance:

- to use the `_` character.
- only one consecutive underscore is allowed.
- only between digits (not allowed at the beginning or end of literals).

Grammar Draft

```
NumericLiteralSeparator ::
  _

DecimalDigits ::
  DecimalDigit
  DecimalDigit NumericLiteralSeparator DecimalDigit
  DecimalDigits DecimalDigit

BinaryDigits ::
  BinaryDigit
  BinaryDigit NumericLiteralSeparator BinaryDigit
  BinaryDigits BinaryDigit  

OctalDigits ::
  OctalDigit
  OctalDigit NumericLiteralSeparator OctalDigit
  OctalDigits OctalDigit    

HexDigits ::
  HexDigit
  HexDigit NumericLiteralSeparator HexDigit
  HexDigits HexDigit  
```

### Standard library

We want to make sure that this syntax is consistent with the usage of the standard library. This will involve making libraries like the following compatible with the proposed parsing rules:

- Number()
- parseInt()


## Alternative Syntax

Our strawnman strategy is to **start with** a more restrictive rule (i.e. disallow both idioms) and losen it upon later if needed (as opposed to starting more broadly and worrying about backwards compatibility trying to tighten it up later).

In addition to that, we couldn't find good/practical evicence where (a) multiple consecutive underscores or (b) underscores before/after numbers are used effectively, so we chose to leave that addition to a later stage if needed/desired.

### Considerations

The main considerations as we look into [other languages](#References) are:

- should we allow multiple separators (e.g. enforcing 10_000 or allowing 10_________000)?
- what are the restrictions on location (e.g. head/tail allowed _100? or does it need to be between numbers 10_000_000?)?
- which separator digit to use (e.g. 1_000, 1,000 , 1 000)?

Common rules available in other languages are:

* Multiple consecutive underscore allowed and only between digits
* Multiple consecutive underscore allowed, in most positions except for the start of the literal or special positions like a decimal point.
* Only every other N digits (e.g. N = 3 for decimal literals or 4 for hexadecimal ones)


### Character

More work needs to be done to determine the feasibility and desirability of using different characters. As a reference, most languages use `_` (C++ being the notable exception to use `'`), so `_` is a reasonable starting point.

Here are some characters that should be looked at to assess feasibility (i.e. is it gramatically possible?) and desirability (e.g. does it lead to a more readable code?):

- `_` (Java, Python, Perl, Ruby, Rust, Julia, Ada, C#)
- `'` (C++)
- ` `
- `.`

## Acknowledgements

This strawnman proposal was developed with @ajklein and @domenic.

## Building the spec:

npm run-script build


## References


### Prior art

* [Java7](https://docs.oracle.com/javase/7/docs/technotes/guides/language/underscores-literals.html): multiple, only between digits.

```java
long creditCardNumber = 1234_5678_9012_3456L;
long socialSecurityNumber = 999_99_9999L;
float pi = 	3.14_15F;
long hexBytes = 0xFF_EC_DE_5E;
long hexWords = 0xCAFE_F00D;
long maxLong = 0x7fff_ffff_ffff_ffffL;
byte nybbles = 0b0010_0101;
long bytes = 0b11010010_01101001_10010100_10010010;
```

Trade-offs:

```java
float pi1 = 3_.1415F;      // Invalid; cannot put underscores adjacent to a decimal point
float pi2 = 3._1415F;      // Invalid; cannot put underscores adjacent to a decimal point
long socialSecurityNumber1
  = 999_99_9999_L;         // Invalid; cannot put underscores prior to an L suffix

int x1 = _52;              // This is an identifier, not a numeric literal
int x2 = 5_2;              // OK (decimal literal)
int x3 = 52_;              // Invalid; cannot put underscores at the end of a literal
int x4 = 5_______2;        // OK (decimal literal)

int x5 = 0_x52;            // Invalid; cannot put underscores in the 0x radix prefix
int x6 = 0x_52;            // Invalid; cannot put underscores at the beginning of a number
int x7 = 0x5_2;            // OK (hexadecimal literal)
int x8 = 0x52_;            // Invalid; cannot put underscores at the end of a number

int x9 = 0_52;             // OK (octal literal)
int x10 = 05_2;            // OK (octal literal)
int x11 = 052_;            // Invalid; cannot put underscores at the end of a number
```


* [C++](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2013/n3499.html): single, between digits (different separator chosen `'`).

```c++
int m = 36'000'000  // digit separators make large values more readable  
```

* [Swift](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/LexicalStructure.html)
```
TODO(goto): find an example.
```

* [Perl](http://perldoc.perl.org/perldata.html#Scalar-value-constructors): multiple, anywhere

```perl
 3.14_15_92          # a very important number
 4_294_967_296       # underscore for legibility
 0xff                # hex
 0xdead_beef         # more hex
```

* [Ruby](http://ruby-doc.org/core-2.3.0/doc/syntax/literals_rdoc.html#label-Numbers): single, only between digits.
```
1_234
```

* [Rust](https://doc.rust-lang.org/reference.html#number-literals): multiple, anywhere.
```
0b1111_1111_1001_0000_i32;         // type i32
1_234.0E+18f64
```

* [Julia](https://docs.julialang.org/en/release-0.4/manual/integers-and-floating-point-numbers/): single, only between digits.

```
julia> 10_000, 0.000_000_005, 0xdead_beef, 0b1011_0010
(10000,5.0e-9,0xdeadbeef,0xb2)
```

* [Ada](http://archive.adaic.com/standards/83lrm/html/lrm-02-04.html#2.4): single, only between digits.
```
123_456
3.14159_26

```

### Ongoing Proposals

* [Python Proposal: Underscore in Numeric Literals](https://www.python.org/dev/peps/pep-0515/#id19): single, only between digits.

```python
# grouping decimal numbers by thousands
amount = 10_000_000.0

# grouping hexadecimal addresses by words
addr = 0xCAFE_F00D

# grouping bits into nibbles in a binary literal
flags = 0b_0011_1111_0100_1110

# same, for string conversions
flags = int('0b_1111_0000', 2)
```

* [C# Proposal: Digit Separators](https://github.com/dotnet/roslyn/issues/216): multiple, only between digits.

```
int bin = 0b1001_1010_0001_0100;
int hex = 0x1b_a0_44_fe;
int dec = 33_554_432;
int weird = 1_2__3___4____5_____6______7_______8________9;
double real = 1_000.111_1e-1_000;
```


### Related Work

* [Format Specifier For Thousands Separator](https://www.python.org/dev/peps/pep-0378/)

