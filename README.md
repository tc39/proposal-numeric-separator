# Numeric Separators

## Stage 4

This is a [proposal](https://tc39.github.io/process-document/), the result of a merge between an earlier draft of itself and Christophe Porteneuve's [proposal-numeric-underscores](https://github.com/tdd/proposal-numeric-underscores), to extend the existing [_NumericLiteral_](https://tc39.github.io/ecma262/#prod-NumericLiteral) to allow a separator character between digits.

## Acknowledgements

This proposal is currently championed by @samuelgoto, @rwaldron, and @leobalter.

This proposal was originally developed by @samuelgoto, @ajklein, @domenic, @rwaldron and @tdd.

## Motivation

This feature enables developers to make their numeric literals more readable by creating a visual separation between groups of digits. Large numeric literals are difficult for the human eye to parse quickly, especially when there are long digit repetitions. This impairs both the ability to get the correct value / order of magnitude...

```js
1000000000   // Is this a billion? a hundred millions? Ten millions?
101475938.38 // what scale is this? what power of 10?
```

...but also fails to convey some use-case information, such as fixed-point arithmetic using integers.  For instance, financial computations often work in 4- to 6-digit fixed-point arithmetics, but even storing amounts as cents is not immediately obvious without separators in literals:

```js
const FEE = 12300;
// is this 12,300? Or 123, because it's in cents?

const AMOUNT = 1234500;
// is this 1,234,500? Or cents, hence 12,345? Or financial, 4-fixed 123.45?
```
Using underscores (`_`, U+005F) as separators helps improve readability for numeric literals, both integers and floating-point (and in JS, it's all floating-point anyway):
```js
1_000_000_000           // Ah, so a billion
101_475_938.38          // And this is hundreds of millions

let fee = 123_00;       // $123 (12300 cents, apparently)
let fee = 12_300;       // $12,300 (woah, that fee!)
let amount = 12345_00;  // 12,345 (1234500 cents, apparently)
let amount = 123_4500;  // 123.45 (4-fixed financial)
let amount = 1_234_500; // 1,234,500
```

Also, this works on the fractional and exponent parts, too:

```js
0.000_001 // 1 millionth
1e10_000  // 10^10000 -- granted, far less useful / in-range...
```

## Examples

(The following examples also appear in the README.md of Babel transform plugin for this proposal.)

### Regular Number Literals

```js
let budget = 1_000_000_000_000;

// What is the value of `budget`? It's 1 trillion!
// 
// Let's confirm:
console.log(budget === 10 ** 12); // true
```

### Binary Literals

```js
let nibbles = 0b1010_0001_1000_0101;

// Is bit 7 on? It sure is!
// 0b1010_0001_1000_0101
//             ^
//
// We can double check: 
console.log(!!(nibbles & (1 << 7))); // true
```

### Hex Literal

```js
// Messages are sent as 24 bit values, but should be 
// treated as 3 distinct bytes:
let message = 0xA0_B0_C0;

// What's the value of the upper most byte? It's A0, or 160.
// We can confirm that:
let a = (message >> 16) & 0xFF; 
console.log(a.toString(16), a); // a0, 160

// What's the value of the middle byte? It's B0, or 176.
// Let's just make sure...
let b = (message >> 8) & 0xFF;
console.log(b.toString(16), b); // b0, 176

// What's the value of the lower most byte? It's C0, or 192.
// Again, let's prove that:
let c = message & 0xFF;
console.log(c.toString(16), b); // c0, 192
```

### BigInt Literal

Numeric Separators are also available within BigInt literals.

```js
// Verifying max signed 64 bit numbers:
const max = 2n ** (64n - 1n) - 1n;
console.log(max === 9_223_372_036_854_775_807n);
```

It can also be used similarly to Number literals

```js
let budget = 1_000_000_000_000n;

// What is the value of `budget`? It's 1 trillion!
// 
// Let's confirm:
console.log(budget === BigInt(10 ** 12)); // true
```

Numeric Separators are only allowed between digits of BigInt literals, and not immediately before the BigInt `n` suffix.

```js
// Valid
1_1n;
1_000n;
99999999_111111111_00000000n;

// Invalid: SyntaxError!
1_n;
0_n;
1000000_n;
1_000_000_n;
```



### Octal Literal

While there isn't much of a benefit, numeric separators are available in the Octal Literal productions out of conventially being generally available in non-legacy productions. In other words, the intent for feature is to be broad available in non-legacy numeric literal types.

```js
let x = 0o1234_5670;
let partA = (x & 0o7777_0000) >> 12; // 3 bits per digit
let partB = x & 0o0000_7777;
console.log(partA.toString(8)); // 1234
console.log(partB.toString(8)); // 5670
```

## Specification

You can see what the specification design looks like [here](spec.md) and a more detailed version [here](https://tc39.github.io/proposal-numeric-separator).

## Background

### Alternative Syntax

Our strawnman strategy is to **start with** a more restrictive rule (i.e. disallow both idioms) and losen it upon later if needed (as opposed to starting more broadly and worrying about backwards compatibility trying to tighten it up later).

In addition to that, we couldn't find good/practical evicence where (a) multiple consecutive underscores or (b) underscores before/after numbers are used effectively, so we chose to leave that addition to a later stage if needed/desired.

### Character

The `_` was agreed to as part of Stage 1 acceptance.
The following examples show numeric separators as they appear in other programming languages:

- **`_` (Java, Python, Perl, Ruby, Rust, Julia, Ada, C#)**
- `'` (C++)

## Building the spec

```sh
npm i
npm run build
```

## References

### Prior art

* [Java7](https://docs.oracle.com/javase/7/docs/technotes/guides/language/underscores-literals.html): multiple, only between digits.

```java
float pi = 	3.14_15F;
long hexBytes = 0xFF_EC_DE_5E;
long hexWords = 0xCAFE_F00D;
long maxLong = 0x7fff_ffff_ffff_ffffL;
byte nybbles = 0b0010_0101;
long bytes = 0b11010010_01101001_10010100_10010010;
```

> Note that the first two examples are actually unlikely to be correct in any circumstance.
Trade-offs:

```java
float pi1 = 3_.1415F;      // Invalid; cannot put underscores adjacent to a decimal point
float pi2 = 3._1415F;      // Invalid; cannot put underscores adjacent to a decimal point

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

```swift
let m = 36_000_000 // Underscores (_) are allowed between digits for readability
```

* [Perl](http://perldoc.perl.org/perldata.html#Scalar-value-constructors): multiple, anywhere

```perl
 3.14_15_92          # a very important number
 4_294_967_296       # underscore for legibility
 0xff                # hex
 0xdead_beef         # more hex
```

* [Ruby](http://ruby-doc.org/core-2.3.0/doc/syntax/literals_rdoc.html#label-Numbers): single, only between digits.

```ruby
1_234
```

* [Rust](https://doc.rust-lang.org/reference.html#number-literals): multiple, anywhere.

```rust
0b1111_1111_1001_0000_i32;         // type i32
1_234.0E+18f64
```

* [Julia](https://docs.julialang.org/en/release-0.4/manual/integers-and-floating-point-numbers/): single, only between digits.

```julia
julia> 10_000, 0.000_000_005, 0xdead_beef, 0b1011_0010
(10000,5.0e-9,0xdeadbeef,0xb2)
```

* [Ada](http://archive.adaic.com/standards/83lrm/html/lrm-02-04.html#2.4): single, only between digits.

```ada
123_456
3.14159_26

```

* [Kotlin](https://kotlinlang.org/docs/reference/basic-types.html#underscores-in-numeric-literals-since-11)

```kotlin
val oneMillion = 1_000_000
val creditCardNumber = 1234_5678_9012_3456L
val socialSecurityNumber = 999_99_9999L
val hexBytes = 0xFF_EC_DE_5E
val bytes = 0b11010010_01101001_10010100_10010010
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

### Implementations

* [Shipping in V8 v7.5 / Chrome 75](https://v8.dev/blog/v8-release-75#numeric-separators)
* [Shipping in SpiderMonkey / Firefox 70](https://bugzilla.mozilla.org/show_bug.cgi?id=1435818)
* [Shipping in JavaScriptCore / Safari 13](https://bugs.webkit.org/show_bug.cgi?id=196351)
