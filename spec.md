# Semantics

This feature is designed to have no impact on the interpretation semantics of numeric literals: `_` are to be ignored by interpreters and should have no effect. They are meant **exclusively** as a visual clue to aid development and have no runtime semantics.

## Syntax

The following grammar represents the Stage 1 criteria, which is: 

1. No leading or trailing separator.
2. No multiple adjacent separator.
3. No separator adjacent to decimal point `.`
4. No separator adjacent to _ExponentIndicator_.
5. No separator adjacent to `0b`, `0B`, `0o`, `0O`, `0x`, `0X`.

Changes to [Numeric Literals](https://tc39.es/ecma262/#prod-NumericLiteral) shown in rendered proposal.

Changes to [ToNumber Applied to the String Type](https://tc39.es/ecma262/#sec-tonumber-applied-to-the-string-type) & [Runtime Semantics: MV](https://tc39.es/ecma262/#sec-runtime-semantics-mv-s) shown in rendered proposal.

Changes to [Runtime Semantics: MV](https://tc39.es/ecma262/#sec-runtime-semantics-mv-s) shown in rendered proposal.

### The Number Constructor, Number ( value )

[**The Number constructor**](https://tc39.es/ecma262/#sec-number-constructor-number-value) and [**Number(value)**](https://tc39.es/ecma262/#sec-number-constructor-number-value) both rely on [**ToNumber**](https://tc39.es/ecma262/#sec-tonumber), so there is no change needed.

### ToNumber ( argument )

The abstract operation [ToNumber ( argument )](https://tc39.es/ecma262/#sec-tonumber) requires no change since it relies on the BinaryIntegerLiteral, OctalIntegerLiteral, HexIntegerLiteral and DecimalDigits changed above.

### ToNumber Applied to the String Type, Runtime Semantics: MV

The syntax defined in [ToNumber Applied to the String Type](https://tc39.es/ecma262/#sec-tonumber-applied-to-the-string-type) is adjusted to use the newly defined _StrDecimalDigits_, preserving the behavior of [parseFloat( string )](https://tc39.es/ecma262/#sec-parsefloat-string). [Runtime Semantics: MV](https://tc39.es/ecma262/#sec-runtime-semantics-mv-s) are also updated by replacing _DecimalDigits_ with _StrDecimalDigits_.

### Global object functions `isFinite` and `isNaN`

Both rely on *ToNumber* semantics (18.2.2 and 18.2.3), so they adjust automatically.

### Global object functions `parseInt` and `parseFloat`

`parseFloat` semantics are unchanged. The syntax for _StrDecimalLiteral_ is updated to define its own _StrDecimalDigits_, preserving the behavior of "parseFloat applied to the String type".

`parseInt` semantics are unchanged.

### Methods of the `Number` constructor

All detection methods (`isXx` methods) operate on actual number values, such as number literals, so they do not introduce extra conversion / parsing rules.  The `parseFloat` and `parseInt` methods are just convenience aliases over their homonyms in the global object.
