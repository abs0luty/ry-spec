# Notation

The syntax is specified using Wirth syntax notation (WSN) which is an alternative to BNF.

> See [Wikipedia](https://en.wikipedia.org/wiki/Wirth_syntax_notation) for more information.

# Characters

<pre>
<a id="unicode_except_newline">unicode_except_newline</a> = /* arbitrary Unicode code point except newline */ .
<a id="unicode_letter">unicode_letter</a> = /* Unicode code point in one of: Lu, Ll, Lt, Lm, or Lo categories */ .
</pre>

The <a href="#unicode_except_newline">unicode_except_newline</a> character definition represents any arbitrary Unicode code point except for the newline character. This definition is used for example in string or character literals to represent character values **except** newline.

The <a href="#unicode_letter">unicode_letter</a> character definition represents Unicode code points that belong to one of the following categories: Lu (Uppercase Letter), Ll (Lowercase Letter), Lt (Titlecase Letter), Lm (Modifier Letter), or Lo (Other Letter). This definition includes characters from various scripts and languages that are commonly used as letters in human-written text.

In Ry, <a  href="#unicode_letter">unicode_letter</a> characters can be used for defining identifiers, variable names, function names, class names, and other similar constructs.

# Letters and digits

<pre>
<a id="id_start">id_start</a> = <a href="#unicode_letter">unicode_letter</a> | "_" .
<a id="id_continue">id_continue</a> = <a href="#unicode_letter">unicode_letter</a> | <a href="#decimal_digit">decimal_digit</a> | "_" .
<a id="binary_digit">binary_digit</a> = "0" | "1" .
<a id="octal_digit">octal_digit</a> = "0" … "7" .
<a id="decimal_digit">decimal_digit</a> = "0" … "9" .
<a id="hexadecimal_digit">hexadecimal_digit</a> = "0" … "9" | "a" … "f" | "A" … "F" .
</pre>

The <a href="#id_start">id_start</a> character definition represents the starting character of any identifier in Ry.

In Ry, identifiers are used to represent names of variables, functions, classes, or other entities. An identifier must begin with a character defined by <a href="#id_start">id_start</a> followed by zero or more characters defined by id_continue. And so the <a href="#id_continue">id_continue</a> character definition represents the continuing characters of an identifier in Ry. See [Identifiers section](#identifiers) for more information.

<a href="#binary_digit">binary_digit</a>, <a href="#octal_digit">octal_digit</a>, <a href="#hexadecimal_digit">hexadecimal_digit</a>, as well as <a href="#decimal_digit">decimal_digit</a> are used in numeric literals.

These character definitions play a crucial role in determining the valid characters that can be used in identifiers and numeric literals within the Ry programming language.

# Lexical elements

This section contains information for implementing the lexing stage of compilation.

## Identifiers

Identifiers name program entities such as variables, traits, types, etc. A "name" is a token, that starts with <a href="#id_start">id_start</a> and then continues with character that satisfies <a href="#id_continue">id_continue</a>. But: not every "name" token is an identifier. Identifier is a name token that is not one of the **reserved keywords**, which are listed below (see [Keywords section](#keywords)).

<pre>
<a>name</a> = <a href="#id_start">id_start</a> { <a href="#id_continue">id_continue</a> } . 
<a>identifier</a> = /* every "name" except <a href="#keyword">keyword</a> */ . 
</pre>

Here are some examples of valid identifiers:
```
a
_foo
ThisIsAlsoValidIdentifier
привет_мир
αβ
你好
```

The implementation is also recommended to normalize all identifiers in the source code using Normalization Form C (NFC) as defined in [Unicode Standart Annex #15](http://www.unicode.org/reports/tr15/).

## Keywords

The following keywords are reserved and may not be used as identifiers.

<pre>
<a id="keyword">keyword</a> = "as" | "break" | "continue" | "enum" | "else" | "for" | "fun" | "if" | "import" | "impl" | "struct" | "trait" | "let" | "match" | "where" | "pub" | "true" | "false" | "return" .
</pre>

## Integer literals

An integer literal is a sequence of digits representing a constant. An optional prefix sets a non-decimal base: `0b` or `0B` for binary, `0o` or `0O` for octal, and `0x` or `0X` for hexadecimal.

A single `0` is considered a decimal zero. In hexadecimal literals, letters `a` through `f` and `A` through `F` represent values `10` through `15`. 

For readability, an underscore character _ may appear after a base prefix or between successive digits. Such underscores do not change the literal's value.

<pre>
<a id="integer_literal">integer_literal</a> = <a href="#binary_literal">binary_literal</a> | <a href="#octal_literal">octal_literal</a> | <a href="#decimal_literal">decimal_literal</a> | <a href="#hexadecimal_literal">hexadecimal_literal</a> .

<a id="binary_literal">binary_literal</a> = "0" ( "b" | "B" ) [ "_" ] <a href="#binary_digits">binary_digits</a> .
<a id="binary_digits">binary_digits</a> = <a href="#binary_digit">binary_digit</a> { [ "_" ] <a href="#binary_digit">binary_digit</a> } .

<a id="octal_literal">octal_literal</a> = "0" ( "o" | "O" ) [ "_" ] <a>octal_digits</a> .
<a id="octal_digits">octal_digits</a> = <a href="#octal_digit">octal_digit</a> { [ "_" ] <a href="#octal_digit">octal_digit</a> } .

<a id="decimal_literal">decimal_literal</a> = "0" | ( "1" … "9" ) [ "_" ] <a>decimal_digits</a> .
<a id="decimal_digits">decimal_digits</a> = <a href="#decimal_digit">decimal_digit</a> { [ "_" ] <a href="#decimal_digit">decimal_digit</a> } .

<a id="hexadecimal_literal">hexadecimal_literal</a> = "0" ( "x" | "X" ) [ "_" ] <a>hexadecimal_digits</a> .
<a>hexadecimal_digits</a> = <a href="#hexadecimal_digit">hexadecimal_digit</a> { [ "_" ] <a href="#hexadecimal_digit">hexadecimal_digit</a> } .
</pre>

Here are some examples of valid and invalid integer literals:
```
0b1010
0b1100
0b10_11_00
0b_10_11_00
0b // empty integer literal
0b123 // unexpected `2`
0o17
0o // empty integer literal
0o88 // unexpected `8`
0xFF
0xABC
OxF_E_D_C
0x // empty integer literal
0xG1 // unexpected `G`
0x1_2_3_4_
9_ // `_` must separate successive digits
9__2 // only one `_` at a time
0_x9 // `_` must separate successive digits
```

## Floating-point literals

A floating-point literal is a representation of a floating-point constant.

A floating-point literal consists of an integer part (sequence of decimal digits), a decimal point, a fractional part (sequence of decimal digits), and an exponent part (`e` or `E` followed by an optional sign (`+` or `-`) and sequence of decimal digits).

<pre>
<a id="float_literal">float_literal</a> = <a href="#decimal_digits">decimal_digits</a> "." [ <a href="#decimal_digits">decimal_digits</a> ] [ <a href="#exponent">exponent</a> ] 
                     | <a href="#decimal_digits">decimal_digits</a> <a href="#exponent">exponent</a> 
                     | "." <a href="#decimal_digits">decimal_digits</a> [ <a href="#exponent">exponent</a> ] .
<a id="exponent">exponent</a> = ( "e" | "E" ) [ "+" | "-" ] <a href="#decimal_digits">decimal_digits</a> .
</pre>

One of the integer part or the fractional part may be elided.

```
.293 // ok
10.  // ok
```

One of the decimal point or the exponent part may be elided:
```
11.3e2 // floating-point literal (decimal point and exponent part are both not elided)
2e9    // floating-point literal (decimal point elided)
2.     // floating-point literal (exponent part elided)
2      // integer literal (both are elided)
```

An exponent value, which is a sequence of digits + their optional sign (`+` by default) scales the mantissa by 10<sup>exp</sup>.

Here are some more examples of valid floating-point literals:
```
0.
93.12
9.113
012.3     // == 12.3
1.e+0
1394.e81-2
1.E-0
1_2.      // == 12.0
0.21e+0_2 // == 21.0
```

## Character literals

In the Ry programming language, a character literal is represented as one or more characters enclosed in single quotes. Within the quotes, any character can appear except for a newline and an unescaped single quote. A single quoted character represents the Unicode value of the character itself.

The character literal in Ry can be a single character, and multiple UTF-8-encoded bytes may represent a single integer value. For instance,`'ä'` holds two bytes (`0xc3 0xa4`) representing the literal 'a-dieresis', `Unicode U+00E4`, with a value of `0xe4`.

In Ry, there are several backslash escapes that allow representing arbitrary values as ASCII text. These escapes include '\x' followed by exactly two hexadecimal digits, '\u' followed by exactly four hexadecimal digits, '\U' followed by exactly eight hexadecimal digits, and '', followed by exactly three octal digits. The value of the literal is determined by the digits in the corresponding base.

Additionally, after a backslash, certain single-character escapes represent special values:

- `\a` represents the alert or bell character (`U+0007`)
- `\b` represents the backspace (`U+0008`)
- `\f` represents the form feed character (`U+000C`)
- `\v` represents the vertical tab character (`U+005C`)
- `\n` represents the line feed or newline character (`U+000A`)
- `\t` represents the horizontal tab character (`U+0009`)
- `\r` represents the carriage return character (`U+000D`)
- `\\` represents the backslash character (`U+005C`)
- `\'` represents the single quote character (`U+0027`), valid only within character literals
- `\"` represents the double quote character (`U+0022`), valid only within string literals

An unrecognized character following a backslash in a character literal is considered illegal.

<pre>
<a id="character_literal">character_literal</a> = "'" <a href="#character">character</a> "'" .
<a id="character">character</a> = <a href="#byte_value">byte_value</a> | <a href="#unicode_value">unicode_value</a> .
<a id="unicode_value">unicode_value</a> = <a href="#characters">unicode_except_newline</a> | <a href="#little_u_value">little_u_value</a> | <a href="#big_u_value">big_u_value</a> | <a href="#escaped_character">escaped_character</a> .
<a id="#escaped_character">escaped_character</a> = `\` ( "a" | "b" | "f" | "n" | "r" | "t" | "v" | `\` | "'" | `"` ) .
<a id="little_u_value">little_u_value</a> = `\` "u" "{" <a href="#hexadecimal_digit">hexadecimal_digit</a> <a href="#hexadecimal_digit">hexadecimal_digit</a> <a href="#hexadecimal_digit">hexadecimal_digit</a> <a href="#hexadecimal_digit">hexadecimal_digit</a> "}" .
<a id="big_u_value">big_u_value</a> = `\` "U" "{" <a href="#hexadecimal_digit">hexadecimal_digit</a> <a href="#hexadecimal_digit">hexadecimal_digit</a> <a href="#hexadecimal_digit">hexadecimal_digit</a> <a href="#hexadecimal_digit">hexadecimal_digit</a> <a href="#hexadecimal_digit">hexadecimal_digit</a> <a href="#hexadecimal_digit">hexadecimal_digit</a> <a href="#hexadecimal_digit">hexadecimal_digit</a> <a href="#hexadecimal_digit">hexadecimal_digit</a> "}" .
<a id="byte_value">byte_value</a> = `\` "x" "{" <a href="#hexadecimal_digit">hexadecimal_digit</a> <a href="#hexadecimal_digit">hexadecimal_digit</a> "}" .
</pre>

Here are some examples of valid and invalid character literals:

```
'a'
'ä'
'ю'
'\x{ff}'
'\x{07}'
'\u{12e4}'
'\U{U00101234}'
```

## String literals

String literals are character sequences between double quotes, as in`"foo"`. Within the quotes, any character may appear except newline and unescaped double quote. The text between the quotes forms the value of the literal, with backslash escapes interpreted as they are in character literals (except that `\'` is illegal and `\"` is legal), with the same restrictions. The two-digit hexadecimal (\x{nn}) escapes represent individual bytes of the resulting string; all other escapes represent the (possibly multi-byte) UTF-8 encoding of individual characters. Thus inside a string literal \x{FF} represents a single byte of value `0xFF`=`255`, while `ÿ`, `\u{00FF}`, `\U{000000FF}` and `\x{c3}\x{bf}` represent the two bytes `0xc3 0xbf` of the UTF-8 encoding of character `U+00FF`.

<pre>
<a>string_literal</a> = `"` { <a href="#character">character</a> } `"` .
</pre>

Here are some examples of string literals:
```
"Hello, world!"
"Привет, мир!"
"日"
"\U{65e5}"           // 日
"\x{e6}\x{97}\x{a5}" // 日
```

# Syntax - Items

TODO: everything.

## Function item
<pre>
<a>FunctionItem</a> = [ "pub" ] "fun" <a href="#identifiers">identifier</a> [ <a>Generics</a> ] "(" <a>FunctionParameters</a> ")" [ ":" <a>Type</a> ] [ <a>WhereClause</a> ] ( ";" | <a>Block</a> ) .
</pre>
