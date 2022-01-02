---
Go语言规范字面量元素
---

[address]: https://go.dev/ref/spec#Lexical_elements

[TOC]



### 1注释

Comments【注释】 serve as program documentation. There are two forms:

1. *Line comments* start with the character sequence `//` and stop at the end of the line.行注释
2. *General comments* start with the character sequence `/*` and stop with the first subsequent character sequence `*/`.普通注释

A comment cannot start inside a [rune](https://go.dev/ref/spec#Rune_literals) or [string literal](https://go.dev/ref/spec#String_literals), or inside a comment. 

注释不能以rune或者字符串字面量开始，也不可以以一个注释开始  也就是 不能 // // 或者// /*

A general comment containing no newlines acts like a space. Any other comment acts like a newline.

一个`general`的注释 不包含 newline，这一个整个注释的在源码中的作用和`space`一样，只是一个空格 。其他的（也就是包含Newline的`general`）作用和newline一样。



### 2Tokens令牌

#### token含义

Tokens form the vocabulary of the Go language. 构成GoLan的词汇表

#### !!!五类token：标识符，关键字，运算符，标点，字面量

There are four classes: *identifiers*, *keywords*, *operators and punctuation*【标点】, and *literals*.

####  被忽略的token：whiteSpace

*White space*, formed from spaces (U+0020), horizontal tabs (U+0009), carriage returns (U+000D), and newlines (U+000A), is ignored except as it separates tokens that would otherwise combine into a single token. 由空格 (U+0020)、水平制表符 (U+0009)、回车符 (U+000D) 和换行符 (U+000A) 组成的空白被忽略，除非它分隔了原本会组合成单个的标记 令牌。  

Also, a newline or end of file may trigger the insertion of a [semicolon](https://go.dev/ref/spec#Semicolons).此外，换行符或文件结尾可能会触发分号的插入。 

While breaking the input into tokens, the next token is the longest sequence of characters that form a valid token.

 在将输入分解为标记时，下一个标记是形成有效标记的最长字符序列。

### 3Semicolons分号

#### 3.1分号作用：terminator

The formal grammar uses semicolons `";"` as terminators in a number of productions. 

正式语法使用分号“;” 作为许多`production`的终结者。

[关于production的定义]: https://go.dev/ref/spec#Notation

#### 3.2忽略分号的2种情况：

Go programs may omit most of these semicolons using the following two rules:

1. When the` input` is broken into tokens, a semicolon is automatically inserted into the token stream immediately after a line's final token if that token is
   
   ##### 疑问：
   
   ##### input是？我们敲进去的代码？分号是编译器自动插入的？
   
   输入分解成token之后，一个分号会立即自动插入在`token流` 紧跟在一·`line`的最后的token 如果这个token是以下....
   
   - `an [identifier](https://go.dev/ref/spec#Identifiers) 标识符
   
   - an [integer](https://go.dev/ref/spec#Integer_literals), [floating-point](https://go.dev/ref/spec#Floating-point_literals), [imaginary](https://go.dev/ref/spec#Imaginary_literals), [rune](https://go.dev/ref/spec#Rune_literals), or [string](https://go.dev/ref/spec#String_literals) literal  各种字面量
   
   - one of the [keywords](https://go.dev/ref/spec#Keywords) `break`, `continue`, `fallthrough`, or `return` 关键字
   
   - one of the [operators and punctuation](https://go.dev/ref/spec#Operators_and_punctuation) `++`, `--`, `)`, `]`, or `}`   操作符或标点符号
   
     
   
2. To allow complex statements to occupy a single line, a semicolon may be omitted before a closing `")"` or `"}"`.要允许复杂语句占据一行，可以省略分号 在结束“)”或“}”之前。

To reflect idiomatic use, code examples in this document elide semicolons using these rules.为了反映惯用用法，本文档中的代码示例使用这些规则省略了分号。

### 4Identifiers标识符

#### 作用，规范

Identifiers name program entities such as variables and types. An identifier is a sequence of one or more letters and digits. The first character in an identifier must be a letter.

标识符命名程序实体，如变量和类型。标识符是由一个或多个字母和数字组成的序列。标识符的第一个字符必须是字母。

```
identifier = letter { letter | unicode_digit } .
a
_x9
ThisVariableIsExported
αβ
```

Some identifiers are [predeclared](https://go.dev/ref/spec#Predeclared_identifiers).预声明

### 5Keywords

#### 关键字和标识符冲突

The following keywords are reserved and may not be used as identifiers.

```
break        default      func         interface    select
case         defer        go           map          struct
chan         else         goto         package      switch
const        fallthrough  if           range        type
continue     for          import       return       var
```

### 6Operators and punctuation运算符标点

The following character sequences represent [operators](https://go.dev/ref/spec#Operators) (including [assignment operators](https://go.dev/ref/spec#assign_op)【赋值运算符】) and punctuation:

```
+    &     +=    &=     &&    ==    !=    (    )
-    |     -=    |=     ||    <     <=    [    ]
*    ^     *=    ^=     <-    >     >=    {    }
/    <<    /=    <<=    ++    =     :=    ,    ;
%    >>    %=    >>=    --    !     ...   .    :
     &^          &^=
```

### 7Integer literals整型字面量

##### 字面量构成

An integer literal is a sequence of digits representing an [integer constant](https://go.dev/ref/spec#Constants). 

整数文字量是表示整数常量的数字序列。

##### 可选前缀

An optional prefix sets a non-decimal base: `0b` or `0B` for binary, `0`, `0o`, or `0O` for octal, and `0x` or `0X` for hexadecimal. 

一个可选的前缀设置一个非十进制的基数: `0B` 或`0b` 表示二进制，`0`,`0O `或`0o` 表示八进制，`0X` 或`0x` 表示十六进制。

A single `0` is considered a decimal zero. 一个0被认为是一个十进制零

In hexadecimal literals, letters `a` through `f` and `A` through `F` represent values 10 through 15.在十六进制字面值中，字母 a 到 f 和 a 到 f 表示值10到15。

##### 下划线：1出现位置有限制；2不改变字面量

For readability, an underscore character `_` may appear after a base prefix or between successive digits; such underscores do not change the literal's value.为了便于阅读，下划线字符 _ 可以出现在基前缀之后或连续数字之间; 这样的下划线不会改变字面值。

##### 字面量表示

```
字面量  4种类型 10进制字面量，2进制，8进制，16进制
int_lit        = decimal_lit | binary_lit | octal_lit | hex_lit .
decimal_lit    = "0" | ( "1" … "9" ) [ [ "_" ] decimal_digits ] .
                   要么是a single `0` 要么是 1~9 可选的 下划线  可选的 10进制数字
binary_lit     = "0" ( "b" | "B" ) [ "_" ] binary_digits .
                前缀 0b或0B 可选下划线，二进制数字
octal_lit      = "0" [ "o" | "O" ] [ "_" ] octal_digits .
               
hex_lit        = "0" ( "x" | "X" ) [ "_" ] hex_digits .
```



```
The underscore character `_` (U+005F) is considered a letter.
下划线字符 _ (u + 005f)被认为是字母。
letter        = unicode_letter | "_" .
decimal_digit = "0" … "9" .
binary_digit  = "0" | "1" .
octal_digit   = "0" … "7" .
hex_digit     = "0" … "9" | "A" … "F" | "a" … "f" .

```

[^digit含义链接]: https://go.dev/ref/spec#Letters_and_digits

```
decimal_digits = decimal_digit { [ "_" ] decimal_digit } .
binary_digits  = binary_digit { [ "_" ] binary_digit } .
octal_digits   = octal_digit { [ "_" ] octal_digit } .
hex_digits     = hex_digit { [ "_" ] hex_digit } .
一样的·
42
4_2 

0600
0_600

0o600
0O600  // second character is capital letter 'O' 大写字母0

0xBadFace
0xBad_Face

0x_67_7a_2f_cc_40_c6

170141183460469231731687303715884105727
170_141183_460469_231731_687303_715884_105727

```

##### 非法字面量：

```
_42         // an identifier, not an integer literal
42_         // invalid: _ must separate successive digits看限制
4__2        // invalid: only one _ at a time只能用一次
0_xBadFace  // invalid: _ must separate successive digits 分隔连续数字 不影响值，这都改变前缀了
```



### 8Floating-point literals

A floating-point literal is a decimal or hexadecimal representation of a [floating-point constant](https://go.dev/ref/spec#Constants).浮点文字是浮点常数的十进制或十六进制表示形式。

#### 一.10进制浮点小数

##### 1构成

A decimal floating-point literal consists of an integer part (decimal digits), a decimal point, a fractional part (decimal digits), and an exponent part (`e` or `E` followed by an optional sign and decimal digits).  

##### 2省略的表达：

One of the integer part or the fractional part may be elided; 2选一省略

one of the decimal point or the exponent part may be elided. 二选一省略

An exponent value exp scales the mantissa (integer and fractional part) by 10<sup>exp</sup>.

十进制浮点数字面值由一个整数部分(用`decimal digits`表示)、一个小数部分(`decimal digits`表示)和一个指数部分(e 或 e 后面跟随一个可选的符号和`decimal digits`)组成。一个整数部分或小数部分可以省略，一个小数点或指数部分可以省略。指数值 exp 将尾数(整数部分和小数部分)缩放10<sup>exp</sup>。

###### 正负号决定是缩还是放



#### 二.16进制浮点小数

##### 1构成·

A hexadecimal floating-point literal consists of a `0x` or `0X` prefix, an integer part (hexadecimal digits), a radix point, a fractional part (hexadecimal digits), and an exponent part (`p` or `P` followed by an optional sign and decimal digits). 

##### 2省略的表达：指数部分一定不能省略

One of the integer part or the fractional part may be elided; the radix point may be elided as well, **but the exponent part is required.** (This syntax matches the one given in IEEE 754-2008 §5.12.3.) An exponent value exp scales the mantissa (integer and fractional part) by 2<sup>exp</sup>.

十六进制浮点数字面值由一个0 x 或0 x 前缀、一个整数部分(十六进制数字)、一个基数部分、一个小数部分(十六进制数字)和一个指数部分(p 或 p 后跟一个可选的符号和十进制数字)组成。可以省略整数部分或小数部分之一; 也可以省略基数点，但需要指数部分。(此语法与 IEEE 754-20085.12.3中给出的语法相匹配。)指数值 exp 将尾数(整数部分和小数部分)缩放2<sup>exp</sup>。

#### 下划线不影响值，有限制

For readability, an underscore character `_` may appear after a base prefix or between successive digits; such underscores do not change the literal value.

#### 字面量表达式

##### 正负号决定是缩还是放

```
float_lit         = decimal_float_lit | hex_float_lit .

decimal_float_lit = decimal_digits "." [ decimal_digits ] [ decimal_exponent ] |
                 方式 1  十进制数字. [小数部分:十进制数字][十进制指数]
                    decimal_digits decimal_exponent |
                 方式 2  十进制数字  十进制指数    省略了 . 和小数部分
                    "." decimal_digits [ decimal_exponent ] .
                 方式 3 . 十进制 数字 [十进制指数]    省略了 整数部分，指数部分可选
decimal_exponent  = ( "e" | "E" ) [ "+" | "-" ] decimal_digits .
                    前缀e或E [正负号]十进制数字

hex_float_lit     = "0" ( "x" | "X" ) hex_mantissa hex_exponent .
hex_尾数
hex_mantissa      = [ "_" ] hex_digits "." [ hex_digits ] |
                    [ "_" ] hex_digits |
                    "." hex_digits .
hex_exponent      = ( "p" | "P" ) [ "+" | "-" ] decimal_digits .
正负号决定是缩还是放
0.
72.40
072.40       // == 72.40
2.71828
1.e+0
6.67428e-11
1E6
.25
.12345E+5
1_5.         // == 15.0
0.15e+0_2    // == 15.0

0x1p-2       // == 0.25
0x2.p10      // == 2048.0
0x1.Fp+0     // == 1.9375
0X.8p-0      // == 0.5
0X_1FFFP-16  // == 0.1249847412109375
0x15e-2      // == 0x15e - 2 (integer subtraction)

0x.p1        // invalid: mantissa has no digits
1p-2         // invalid: p exponent requires hexadecimal mantissa
0x1.5e-2     // invalid: hexadecimal mantissa requires p exponent
1_.5         // invalid: _ must separate successive digits
1._5         // invalid: _ must separate successive digits
1.5_e1       // invalid: _ must separate successive digits
1.5e_1       // invalid: _ must separate successive digits
1.5e1_       // invalid: _ must separate successive digits
```

### 9Imaginary literals

An imaginary literal represents the imaginary part of a [complex constant](https://go.dev/ref/spec#Constants). It consists of an [integer](https://go.dev/ref/spec#Integer_literals) or [floating-point](https://go.dev/ref/spec#Floating-point_literals) literal followed by the lower-case letter `i`. The value of an imaginary literal is the value of the respective integer or floating-point literal multiplied by the imaginary unit *i*.

```
imaginary_lit = (decimal_digits | int_lit | float_lit) "i" .
```

For backward compatibility, an imaginary literal's integer part consisting entirely of decimal digits (and possibly underscores) is considered a decimal integer, even if it starts with a leading `0`.

```
0i
0123i         // == 123i for backward-compatibility
0o123i        // == 0o123 * 1i == 83i
0xabci        // == 0xabc * 1i == 2748i
0.i
2.71828i
1.e+0i
6.67428e-11i
1E6i
.25i
.12345E+5i
0x1p-2i       // == 0x1p-2 * 1i == 0.25i
```

### 10Rune literals直接看表达式就行

A rune literal represents a [rune constant](https://go.dev/ref/spec#Constants), an integer value identifying a Unicode code point. 表示一个 rune 常量，一个整数值表示一个 Unicode码点。

#### rune是什么:单引号括起来的

A rune literal is expressed as one or more characters enclosed in single quotes, as in `'x'` or `'\n'`. Within the quotes, any character may appear except `newline` and `unescaped single quote`.一个 rune literal 一个 rune literal 是用单引号括起来的一个或多个字符来表示的，比如在‘ x’或者‘ n’中。在引号中，除了换行符和未转义的单引号外，任何字符都可以出现。

#### 单字符和多字符

 A single quoted character represents the Unicode value of the character itself, while multi-character sequences beginning with a backslash encode values in various formats.

单个被引号括起来的字符代表他自身的Unicode值，多字符序列是以各种形式的 反斜杠开头的多种形式的编码值

The simplest form represents the single character within the quotes; since Go source text is Unicode characters encoded in UTF-8, multiple UTF-8-encoded bytes may represent a single integer value. For instance, the literal `'a'` holds a single byte representing a literal `a`, Unicode U+0061, value `0x61`, while `'ä'` holds two bytes (`0xc3` `0xa4`) representing a literal `a`-dieresis, U+00E4, value `0xe4`.

最简单的形式表示引号中的单个字符; 因为 Go 源文本是 utf-8编码的 Unicode 字符，所以多个 utf-8编码的字节可以表示单个整数值。例如，字面量‘ a’包含一个字节，表示字面量a，或Unicode + 0061，或value 0x61，而‘ ä’包含两个字节(0xc30xa4) ，表示文本 a-dieresis，u + 00 E4，value 0x4。

Several backslash escapes allow arbitrary values to be encoded as ASCII text. There are four ways to represent the integer value as a numeric constant: `\x` followed by exactly two hexadecimal digits; `\u` followed by exactly four hexadecimal digits; `\U` followed by exactly eight hexadecimal digits, and a plain backslash `\` followed by exactly three octal digits. In each case the value of the literal is the value represented by the digits in the corresponding base.

多个反斜杠转义允许将任意值编码为 ASCII 文本。有四种方法可以将整数值表示为一个数字常量: x 后面紧跟两个十六进制数字; u 后面紧跟四个十六进制数字; u 后面紧跟八个十六进制数字，纯斜杠后面紧跟三个八进制数字。在每种情况下，文字的值都是由对应基数中的数字表示的值。

Although these representations all result in an integer, they have different valid ranges. Octal escapes must represent a value between 0 and 255 inclusive. Hexadecimal escapes satisfy this condition by construction. The escapes `\u` and `\U` represent Unicode code points so within them some values are illegal, in particular those above `0x10FFFF` and surrogate halves.

尽管这些表示都是整数，但它们有不同的有效范围。八进制转义必须表示一个介于0和255之间的值。十六进制转义通过构造来满足这一条件。Escapes u 和 u 表示 Unicode 代码点，因此在它们中的一些值是非法的，特别是那些上面的0x10f 和代理 half。

After a backslash, certain single-character escapes represent special values:

在反斜杠之后，某些单字符转义表示特殊的值:

```
\a   U+0007 alert or bell
\b   U+0008 backspace
\f   U+000C form feed
\n   U+000A line feed or newline
\r   U+000D carriage return
\t   U+0009 horizontal tab
\v   U+000B vertical tab
\\   U+005C backslash
\'   U+0027 single quote  (valid escape only within rune literals)
\"   U+0022 double quote  (valid escape only within string literals)
```

All other sequences starting with a backslash are illegal inside rune literals.

所有其他以反斜杠开头的序列在符文文本中都是非法的。

```
rune_lit         = "  '  " ( unicode_value | byte_value ) "  '  "  .
单引号括起来 unicode值或字节值
                        unicode_value    = unicode_char | little_u_value | big_u_value | escaped_char .
                                           直接是字符
byte_value       = octal_byte_value | hex_byte_value .
                  octal_byte_value = `\` octal_digit octal_digit octal_digit . 3位8进制数字
                  \377
                  hex_byte_value   = `\` "x" hex_digit hex_digit .  2位16进制数字
                  \xFF
                        little_u_value   = `\` "u" hex_digit hex_digit hex_digit hex_digit .
                                \u 4位16进制 \u3333
                        big_u_value      = `\` "U" hex_digit hex_digit hex_digit hex_digit
                                      hex_digit hex_digit hex_digit hex_digit .
                                \U8位16进制 \U00003333
                       escaped_char     = `\` ( "a" | "b" | "f" | "n" | "r" | "t" | "v" | `\` | "'" | `"` ) 
'a'
'ä'
'本'
'\t'
'\000'
'\007'
'\377'
'\x07'
'\xff'
'\u12e4'
'\U00101234'
'\''         // rune literal containing single quote character
'aa'         // illegal: too many characters
'\xa'        // illegal: too few hexadecimal digits
'\0'         // illegal: too few octal digits
'\uDFFF'     // illegal: surrogate half
'\U00110000' // illegal: invalid Unicode code point
```

### 11String literals表达式参考rune

A string literal represents a [string constant](https://go.dev/ref/spec#Constants) obtained from concatenating a sequence of characters. There are two forms: raw string literals and interpreted string literals.

字符串字面值表示串联一个字符序列而获得的字符串常量。有两种形式: 原始字符串和解释字符串。

#### 字面值两种形式

##### 1raw反引号 不是rune的单引号

Raw string literals are character sequences between back quotes, as in ``foo``. Within the quotes, any character may appear except back quote. The value of a raw string literal is the string composed of the uninterpreted (implicitly UTF-8-encoded) characters between the quotes; in particular, backslashes have no special meaning and the string may contain newlines. Carriage return characters ('\r') inside raw string literals are discarded from the raw string value.

原始字符串文字是反引号之间的字符序列，如‘ foo’中的字符序列。在引号中，除了反引号外，任何字符都可以出现。原始字符串的值是由引号之间未解释的(隐式 utf-8编码的)字符组成的字符串; 特别是反斜杠没有特殊含义，字符串可能包含换行符。原始字符串中的回车字符(’r’)将从原始字符串值中丢弃。

##### 2interpreted双引号

Interpreted string literals are character sequences between double quotes, as in `"bar"`. Within the quotes, any character may appear except newline and unescaped double quote. The text between the quotes forms the value of the literal, with backslash escapes interpreted as they are in [rune literals](https://go.dev/ref/spec#Rune_literals) (except that `\'` is illegal and `\"` is legal), with the same restrictions. The three-digit octal (`\`*nnn*) and two-digit hexadecimal (`\x`*nn*) escapes represent individual *bytes* of the resulting string; all other escapes represent the (possibly multi-byte) UTF-8 encoding of individual *characters*. Thus inside a string literal `\377` and `\xFF` represent a single byte of value `0xFF`=255, while `ÿ`, `\u00FF`, `\U000000FF` and `\xc3\xbf` represent the two bytes `0xc3` `0xbf` of the UTF-8 encoding of character U+00FF.

解释的字符串文字是双引号之间的字符序列，如“ bar”中所示。在引号中，除了换行符和未转义的双引号之外，任何字符都可以出现。双引号之间的文本构成了文字的值，反斜杠转义解释为它们是符文文本(除非’是非法的和’是合法的) ，具有相同的限制。三位八进制(nnn)和两位十六进制(xnn)转义表示结果字符串的单个字节; 所有其他转义表示单个字符的(可能是多字节的) utf-8编码。因此，字符串 literal 377和 xFF 表示值0xff = 255的一个字节，而 ÿ、 u00FF、 U000000FF 和 xc3 xbf 表示字符 u + 00ff 的 utf-8编码的两个字节0xc30xbf。

```
string_lit             = raw_string_lit | interpreted_string_lit .
raw_string_lit         = "`" { unicode_char | newline } "`" .
interpreted_string_lit = `"` { unicode_value | byte_value } `"` .
`abc`                // same as "abc"
`\n
\n`                  // same as "\\n\n\\n"转义
"\n"
"\""                 // same as `"`
"Hello, world!\n"
"日本語"
"\u65e5本\U00008a9e"
"\xff\u00FF"
"\uD800"             // illegal: surrogate half
"\U00110000"         // illegal: invalid Unicode code point
```

These examples all represent the same string:

这些例子都代表同一个字符串:

```
"日本語"                                 // UTF-8 input text
`日本語`                                 // UTF-8 input text as a raw literal
"\u65e5\u672c\u8a9e"                    // the explicit Unicode code points
"\U000065e5\U0000672c\U00008a9e"        // the explicit Unicode code points
"\xe6\x97\xa5\xe6\x9c\xac\xe8\xaa\x9e"  // the explicit UTF-8 bytes
```

If the source code represents a character as two code points, such as a combining form involving an accent and a letter, the result will be an error if placed in a rune literal (it is not a single code point), and will appear as two code points if placed in a string literal.

如果源代码将一个字符表示为两个代码点，例如一个包含重音和字母的组合表单，那么如果将其放在一个 rune literal 中，结果将是一个错误(它不是单个代码点) ，如果将其放在一个字符串 literal 中，结果将显示为两个代码点。