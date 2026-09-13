literals-maintainer-notes.md

# what are literals

Built in literals are already defined in Zax like:
```zax
a := 1
b : Integer = 5
c : U8 = 1 << 6
```

There are al kinds of math rules associated with literals.

The fixed floating points and binary values will extend the values:
```
a : MicroFixed16 = 4.5
b : PrecisionFixed64 = 7.25
c : Single = -17.44
d : Half = 1.5 + 1.5
```

Math rules will apply to floating points and then similar to integers the value is expressed into the floating point.

Finally basic ASCII strings exist:
```
a = "hello utf8 string"
b = "goodbye other utf8 string"
```

Normal strings are parsed as utf8 with NO `"\nescape"` encoding. The strings are full utf8 though otherwise aside from a few additional restrictions. They error if the string sequence contains an invalid utf8 sequences, cannot contain any non valid Unicode characters, and cannot contain tab (0x09), line feed (0x0A), carriage returns (0x0D), and double quotes (0x22).

These ranges are banned during all quote parsing:
- 0x00..0x1F
- 0xD800..0xDFFF
- 0xFDD0..0xDFEF    
- 0xFFFE
- 0xFFFF
- 0x1FFFE
- 0x1FFFF
- 0x10FFFE
- 0x10FFFF (and any value beyond this range)

These are not parsable characters and will cause errors if found inside the source file.

That doesn't mean they cannot be represented. This is "normal parsable string behavior" and other sequences are legal.

A `'single quoted'` string is NOT a direct string literal, but it does play a special role when combined with a prefix.

```
// Two physical strings quotes side beside become merged into a single string:
text = "hello " "there "    // becomes "hello there"

// And strings with continuations also become merged into a single string:
moreText = "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor " \
           "incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud"
```

Strings (and arrays) have a fixed maximum size of `IndexSize` in length.


# basic literal operator concept

Literals in Zax take the form of:
```
prefix'<encoded value>'
prefix"<encoded value>"
```

All these forms have the same restrictions as normal strings, except if single quote is used (`'` / 0x27) then single quotes are banned because they close the literal. If you use double quotes (`"` / 0x22) then double quotes are banned because they close the sequence. Literals can merge so switching between banned quotes is done by choosing single or double quotes to encode the other. Single quote are legal in double quotes, and double quotes are legal in single quotes.

If `a` and `b` were a prefix then:
```
value1 := a'<encoding part1>' a"<encoding part2>"       // merge together and are parsed as a unified literal value
value2 := a'<encoding part1>' b"<encoding part2>"       // NOT merge together and are parsed as two distinct literal values but might still merge
value3 := (a'<encoding part1>') (a"<encoding part2>")   // parsed as two different literals but might still merge
```

They auto-merge and parsed as a unified value if the prefixes are identical (even across continuations), and they are not separated by brackets (or other separations). Different prefixes are always parsed independently but still might merge.

String literals are Type based `once` functions that are invoked to convert from a string sequence into a value:
```
MyType :: type {
  operator literal 'cstyle' once final : (output : String)(input : String) = { ... }
}

// ...define alias to `cstyle` operator to expose at this scope (tbd how aliases are defined)...

myStringParsed : String = cstyle"this is parsed however you like"

// the prefix can be type and/or namespace prefixed to be specifically selected
myOtherParsedString := MyType.cstyle'this is also legal'
```

These functions are designed to execute at come time and return a constant that created by the compiler. They are NOT a runtime feature.

A literal function only has a `String` as input, and the string sequence is provided by whatever the compiler sees in the string and must exclude disallowed character sequences. But the programmer is free to using their own escape sequences like `\t` and `\n` or whatever they wish.

The return from a literal function can be any constant value, including from constant types, and does not have to be a string.

The intent is that normally a number, character, String, array, or another custom type would be returned.


## outstanding concern related to scalars

Every literal operator function returns a concrete type. For creating number values, being able to define a number and have that assigned to any type is important, just like any other constant number can be assigned to a number type. The trouble is sometimes literary operators want to intentionally return a type that is only compatible with an expression that takes exactly that type.

Other times the programmer might want to return an `Integer` but then allow that to be used in any number where the value is "compatible". In C++ you can use `explicit` to indicate if compatible conversion happens for an expression, or not.

Zax will need something similar. I think the idea should be that return types can be coerced. Scalars should allow automatic coercing when declared they allow it. Maybe a keyword on the return value declaration could be `implicit` or `coercion` or `conversion`. I'm not sure `coercion` is a bit awkward to spell, but it's highly accurate.

I would look for your input here...

## auto-merging of strings / arrays

Another idea is that if two literals that return `String` are parsed side-by-side (without some lhs/rhs expression), they'd be auto-merged after compiled. That could be true of arrays of the same type. That would allow auto-merging of the literals. Two numbers though, or non-array like values would not be merged.

## namespacing

String literals are defined within a namespace, or within a type. They can be defined within a type, then aliased into a namespace. If two literal definitions collide in the same namespace, they are ambiguous and namespacing must be used to resolve the ambiguity.

## build-in literal operators

### Coerced numerical literal operators

```
myBinary : U8 = b'11111111'         // `b` or `b2`; binary number; 01
myOctal : U8 = o'377'               // `o` or `b8`; octal number; 01234567
myHex : U8 = h'FF'                  // `h` or `b16`; hexadecimal number; 0123456789ABCDEF / 0123456789abcdef
myDuo : U16 = duo'1XE'              // `duo` or `b12`; duodecimal number; 0123456789AB / 0123456789ab / 0123456789TE / 0123456789te / 0123456789XE / 0123456789xe
myBase10 : U16 : b10'48281'         // decimal number; 0123456789
myBase32 : U32 : b32'32Y9'          // 0123456789ABCDEFGHJKMNPQRSTVWXYZ / 0123456789abcdefghjkmnpqrstvwxyz (Crockford Base32)
myBase32Alt : U32 : rfc6458'ABC'    // ABCDEFGHIJKLMNOPQRSTUVWXYZ234567 / abcdefghijklmnopqrstuvwxyz234567
myBase36 : U32 : b36'Z3'            // 0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ / 0123456789abcdefghijklmnopqrstuvwxyz
myBase94 : U32 : b94'!9'            // ! " # $ % & ' ( ) * + , - . / 0 1 2 3 4 5 6 7 8 9 : ; < = > ? @ A B C D E F G H I J K L M N O P Q R S T U V W X Y Z [ \ ] ^ _ ` a b c d e f g h i j k l m n o p q r s t u v w x y z { | } ~
myBase62 : U32 : b62'4y'            // 0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz
myBase58 : U32 : b58'Uu1'           // 123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz
myBase64 : U32 : b64'9+/'           // ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/
myBase20 : U32 : b20'4D'            // 0123456789ABCDEFGHIJ / 0123456789abcdefghij
myBase60 : U32 : b60'k9'            // 0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwx
```

### Non-coerced numerical literal operators

I'm not sure we need this... The only reason I can see these being valuable would be if we wanted a constant decimal value to be declared as a fixed type. As literals do not contain expressions (at least without writing a literal operator that parsed expressions), this mean that a value could be fixed to a type and most likely a decimal number in base-10 otherwise the combinations get a bit crazy.

```
myBinary : U8 = u8'255'             // forced U8 type, expressed as decimal
myBinary : U16 = u16'1024'          // forced U16 type, expressed as decimal
myBinary : U32 = u16'991024'        // forced U32 type, expressed as decimal
myBinary : U64 = u64'99102499'      // forced U64 type, expressed as decimal
// ... other types we'd want...
```

But you could just as easily do:
```
myBinary : U64 = (991024 as U64)    // force the type at compile time, not as elegant as a fenced number
                                    // also makes the value seem as if it wasn't a U64 and now it is...
```

There's only slight elegance to using u64'<constant-number>`, and those numbers must convert first before being able to be used in a calculation.

I guess my question is, would the elegance be worth it? It might be, and if it is, what other literal operators might we want to have for the inventory of numbers we have?


### Non-coerced strings and characters

| character identity | string | internal | char encodings | string encodings |
| --- | --- | --- | --- | --- |
| `Byte` | `String` | `U8` | `by` `byc` `byh` | `s` `c` |
| `UChar` | `AsciiString` | `UChar` | `ch` `chc` `chh` | `ascii` |
| `Usc2` | `Usc2String` | `Usc2` | `bmp16` `bmp16c` `bmp16h` | `usc2` |
| `WChar` | `WideString` | `WChar` | `wch` `wchc` `wchh` | `w` |
| `Rune` | `Utf8String` | `U8` | `rune` `u` `uc` `uh` | `utf8` |
| `Rune` | `Utf16String` | `U16` | `rune` `u` `uc` `uh` | `utf16` |
| `Rune` | `Utf32String` | `U32` | `rune` `u` `uc` `uh` | `utf32` |
| `Rune` | `MbcsString` | `U8` + code page | `rune` `u` `uc` `uh` | `mbcs` |
| `U8` | | `U8[]` | `U8` | | `rfc4648` |

String literal operators:
```
myString : String = s"normal string"                        // direct literal parsing with no escapes, explicit version of ""; returns `String`; `U8` internally;
                                                            // single character uses `Byte` identify;
                                                            // join with `Ascii` `Ucs2String` `WideString` (direct `Byte` to possibly wider conversion)
                                                            // join with `Utf8String` `Utf16String` `Utf32String` (`String` sequence "seen as" UTF-8, fails if not valid UTF-8)
                                                            // join with `MbcsString` (`String` sequences "seen as" `MbcsString` code page, but values must be legal or combined legal values);

myAscii : AsciiString = ascii'payload without escapes'      // direct literal parsing with no escapes, ASCII characters only are legal; returns `AsciiString`;
                                                            // `UChar` internally; single character uses `UChar` identity;
                                                            // join with `String` (direct possible narrowing `UChar` to `Byte`) `WideString` (widens `UChar` to `WChar`)
                                                            // join with `Ucs2String` (converts `UChar` to `Usc2`)
                                                            // join with `Utf8String` `Utf16String` `Utf32String` using a UTF encoded sequence of ASCII values;
                                                            // join with `MbcsString` (where ASCII values are legal to encode within code page, fails if outside);

myUtf8 : Utf8String = utf8'payload retained as written'     // direct literal parsing with no escapes, explicit UTF-8 encoded string; returns `Utf8String`; `U8` internally;
                                                            // single character uses `Rune` identity;
                                                            // join with `String` (direct translation) `WideString` (possible widens `Rune` to `WChar`)
                                                            // join with `AsciiString` (valid characters within the ASCII range, fail if outside, narrows `Rune` to UChar`)
                                                            // join with `Ucs2String` (if within the 16-bit range allowed, fails if outside, narrows `Rune` to `Usc2`)
                                                            // join with `Utf8String` `Utf16String` `Utf32String` using a UTF encoded sequence;
                                                            // join with `MbcsString` (where `Rune` is legal within code page, fails if outside);

myRfc4648 := rfc4648'VGhlIHF1aWNrIGJyb3duIGZveC4='          // RFC 4648 base-64 encoding of a byte sequence, output is a `U8` byte array;
                                                            // uses same "join with" semantics as `String`

myXml : Utf8String = xml'John&#39;s Fish &amp; Chips'       // XML string entity processing, returns `Utf8String`;

myCStyle : String = c"normal\nc-style\0string\t\t\t"        // parsed as c-string escaped string, returns `String`;

// all `Byte` values join with `String` `Usc2String` `WideString` with possible widening;
// joins with `Utf8String` `Utf16String` `Utf32String` and encoded as ASCII to UTF-encoded value;
// adding individual `Byte` values to form a pre-encoded UTF sequence is not possible; Use a `U8[]` array to create a manually pre-encoded UTF sequence;
// joins with `MbcsString` by treating the `Byte` as ASCII and success or failure is based on the ASCII value being within the code page;
myByte : Byte = by'A'                                       // parsed as single byte with no escapes; returns `Byte` identity type; used for String;
myCByte : Byte = byc'\\'                                    // parsed as single byte with c-escapes; returns `Byte` identity type; use for String;
myCByte : Byte = byh'0A'                                    // parsed as single byte as hex; returns `Byte` identity type; use for String;

// `UChar` only joins with `AsciiString`
myCharacter : UChar = ch'X'                                 // parsed as a single character ASCII with no escapes;
                                                            // returns `UChar` identity type; use with AsciiString;
myCharacter : UChar = chc'\n'                               // parsed as a single character ASCII with c-style escapes;
                                                            // returns `UChar` identity type; use with AsciiString;
myCharacter : UChar = chh'\n'                               // parsed as a hex character ASCII;
                                                            // returns `UChar` identity type; use with AsciiString;

// all `Rune` values join with `String` (as UTF-8 sequence) `WideString` (possible widening) `Utf8String` `Utf16String` `Utf32String` (as UTF encoded sequence)
// join with `Ucs2String` (if within the 16-bit range allowed, fails if outside, narrows `Rune` to `Usc2`)
// join with `MbcsString` (where `Rune` is legal within code page, fails if outside)
myRune1 : Rune = rune'☃'                                    // `rune` or `u`; single unicode character with no escapes; used with `Utf8String` `Utf16String` `Utf32String`;
                                                            // returns `Rune` identity type;
myRune2 : Rune = uc'\0'                                     // single unicode character with c-escapes; used with `Utf8String` `Utf16String` `Utf32String`;
                                                            // returns `Rune` identity type;
myRune3 : Rune = uh'2DF'                                    // single unicode character hex encoded; used with `Utf8String` `Utf16String` `Utf32String`;
                                                            // returns `Rune` identity type;

utf8String : Utf8String = utf8'© Snowman Industries (☃)'    // another explicit example of a UTF-8 encoded string

// All the same "join with" properties as UTF-8
myUtf16 : Utf16String = utf16"using UTF-16 string"          // 16-bit wide character string UTF-16 encoded with surrogates and no escapes;
                                                            // `U16` internally; uses `Rune` characters (with use of surrogates);
                                                            // join with `String` (UTF-8 encoded then direct translation) `WideString` (possible widens `Rune` to `WChar`)
                                                            // join with `AsciiString` (valid characters within the ASCII range, fail if outside, narrows `Rune` to UChar`)
                                                            // join with `Ucs2String` (if within the 16-bit range allowed, fails if outside, narrows `Rune` to `Usc2`)
                                                            // join with `Utf8String` `Utf16String` `Utf32String` using a UTF encoded sequence
                                                            // join with `MbcsString` (where `Rune` is legal within code page, fails if outside);

myUtf32 : Utf32String = utf32"using UTF-32 string"          // 32-bit wide character string UTF-32 encoding and no escapes;
                                                            // `U32` internally; uses `Rune` characters;
                                                            // join with `String` (UTF-8 encoded then direct translation) `WideString` (possible widens `Rune` to `WChar`)
                                                            // join with `AsciiString` (valid characters within the ASCII range, fail if outside, narrows `Rune` to UChar`)
                                                            // join with `Ucs2String` (if within the 16-bit range allowed, fails if outside, narrows `Rune` to `Usc2`)
                                                            // join with `Utf8String` `Utf16String` `Utf32String` using a UTF encoded sequence
                                                            // join with `MbcsString` (where `Rune` is legal within code page, fails if outside);

// all `Ucs2` values join with `WideString` `Utf8String` `Utf16String` `Utf32String`
// join with `MbcsString` (where `Ucs2` is promoted to a `Rune` and is legal within code page, fails if outside)
myUcs1 : Ucs2 = bmp16'f'                                    // USC-2 16-bit wide character with no escapes; use with `Usc2String`; returns `Usc2`;
myUcs2 : Ucs2 = bmp16c'\0'                                  // USC-2 16-bit wide character with c-style escapes; use with `Usc2String`; returns `Usc2`;
myUcs3 : Ucs2 = bmp16h'AB'                                  // USC-2 16-bit wide character with hex encoding; use with `Usc2String`; returns `Usc2`;

myUcs2Str : Ucs2String = ucs2"using UTF-32 string"          // 16 bit wide string with no encoding; uses `U16` internally;
                                                            // join with `WideString`
                                                            // join with `Utf8String` `Utf16String` `Utf32String` (where legal within UTF sequence)
                                                            // join with `MbcsString` (where `Rune` legal within code page, fails outside of code page);

myWideStr : WideString = w"using wide string"               // 32-bit character wide string with no encoding; `WChar` internally; single character uses `WChar` identity;

myMbcs : MbcsString = cp1252"using wide string"             // MBCS encoded string, carries applicable code page in type; internally `U8`;
                                                            // extract as `Rune` characters;
                                                            // accepts `Rune` characters (within code page);
                                                            // code pages supported: 
                                                            // `cp1252` — Western European (Windows Latin‑1)
                                                            // `cp1250` — Central European
                                                            // `cp1251` — Cyrillic
                                                            // `cp1253` — Greek
                                                            // `cp1254` — Turkish
                                                            // `cp1255` — Hebrew
                                                            // `cp1256` — Arabic
                                                            // `cp1257` — Baltic
                                                            // `cp1258` — Vietnamese (SBCS + combining accents)
                                                            // `cp932` — Shift‑JIS (Windows) / Shift‑JIS (name used in standards)
                                                            // `cp936` — GBK (Windows) Chinese (Simplified)
                                                            // `gb2312` (older standard)
                                                            // `cp950` — Big5 (Windows) Chinese (Traditional)
                                                            // `big5` (name used in standards)
                                                            // `cp949` — Unified Hangul Code (UHC)
                                                            // `euckr` (name used in standards)
                                                            // `cp437` — Original IBM PC code page
                                                            // `cp850` — Western European (DOS)
                                                            // `cp852` — Central European (DOS)
                                                            // `cp855` — Cyrillic (DOS)
                                                            // `cp866` — Russian (DOS)
                                                            // `cp857` — Turkish (DOS)
                                                            // `cp858` — Western European with Euro symbol
                                                            // `iso88591` — Western European
                                                            // `iso88592` — Central European
                                                            // `iso88595` — Cyrillic
                                                            // `iso88597` — Greek
                                                            // `iso88598` — Hebrew
                                                            // `iso88599` — Turkish
                                                            // `iso885915` — Western European with Euro

hexString : String = hex"0A0D09"                            // hex byte sequence (2 hex digits per character), returned as `String`;
                                                            // join with `String` `Ascii` `WideString` `Ucs2String` `Utf8String` `Utf16String` `Utf32String` `MbcsString`;

hex8String : Utf8String = rhex"000A000D01DF"                // rune hex byte sequence (4 hex digits per character), returned as `Utf8String`;
                                                            // join with `Utf16String` `Utf32String` `MbcsString` (only where legal in code page);

hexAsciiString : AsciiString = ascii'' hex"0A0D09"          // example `Ascii` joined with `String`

utf8HexString : Utf8String = utf8"hello" hex"0A0D09"        // example `Utf8String` joined with `String`

utf16HexString : Utf16String = utf16"hello" hex"0A0D09"     // example `Utf16String` joined with `String`

utf32HexString : Utf32String = utf32"hello" hex"0A0D09"     // example `Utf16String` joined with `String`

mbcsHexString : MbcsString = cp1252"using wide string" hex"003A2B0A0A"  // example MBCS string joined with `String`
```

Other considerations:
- all types that follow a "joins with" are treated as "rhs (right hand side)", where `type <+> rhs`
- joins (`<+>`) at compile time are checked for legal conversion and error upon failure
- joins that can fail at runtime will return an optional result indicating if the join succeeded or not; joins that can't fail resolve normally;
- all MBCS conversions are within a sub namespace and do not pollute the main name space

## Legal UTF-8 string values

Parsed legal strings can contain all UTF-8 character sequences, including control characters (0x00..0x1F), except:
- 0xD800..0xDFFF

And cannot encode beyond:
- 0x10FFFF

These sequences can be encoded but they are discouraged as they will cause conversion failures to Unicode:
- 0xFDD0..0xDFEF
- 0xFFFE
- 0xFFFF
- 0x1FFFE
- 0x1FFFF
- 0x10FFFE
- 0x10FFFF

UTF-8 is internally expressed as an unsigned 8bit character array.

## Legal UTF-16 string values

Parsed legal strings can contain all UTF-16 character sequences, including control characters (0x00..0x1F), except:
- 0xD800..0xDFFF

And cannot encode beyond:
- 0xFFFF

These values are discouraged as they will cause conversion failures to Unicode:
- 0xFFFE
- 0xFFFF


## Legal UTF-32 string values

All 32 bit character values are technically encodable. What converts to other formats or can be presented is dependent on the target encoding. Conversion functions will block some strings converting to other formats.

## MBCS

Multi-byte character sets (MBCS) are byte sequences as defined within their code page. They can be converted into valid UTF sequences / `Rune` / Unicode.


## Auto joining

When two literals are placed side-by-side their results automatically apply the join `<+>` operator. This is a special operator that (while manually invocable) specially reserved for auto-joining. This is how MBCS is able to join with a `String` type to inject specific values into the MBCS range.

A (`+`) binary plus operator is not used as declarations like `ch'a' ch'b'` might become added together, and that is not a desired behavior. The distinct operator lets characters and numbers reject joining.

## Other Types

While the focus of literal operators is to allow number and string formats and encodings, it is not restricted to those types. Any type can create a compile time literal conversion and their own custom compile time and run time join operators.
