---
layout: post
title: "v8的词法分析"
tags:
    - v8
    - token
    - 词法分析
    - 源代码
description: ""
---

本文将探讨v8是如何对utf8编码的js文件进行词法分析的！

## 简介

v8提供了一个`Scanner`类作为词法分析器，在该类中有这么一个指针：

```cpp
Utf16CharacterStream* stream_;
```

这是个啥玩意？看一下源代码中对这个类的解释：

    // Buffered stream of UTF-16 code units, using an internal UTF-16 buffer.

大概就是说，`Utf16CharacterStream`就是一个__UTF-16代码单元__的缓冲流。

啥是__代码单元__？v8的解释如下：

    // A code unit is a 16 bit value representing either a 16 bit code point
    // or one part of a surrogate pair that make a single 21 bit code point.

这就涉及到了unicode规范，在unicode字符集中，每个unicode字符都有一个唯一的__代码点__。

在utf8、utf16等编码形式中，代码点被映射到一个或者多个代码单元。

代码单元是各个编码形式中的最小单元，其大小：

* UTF-8 中的代码单元由 8 位组成。
* UTF-16 中的代码单元由 16 位组成。

每个代码点需要多少个代码单元呢？不同编码形式不一样：

* UTF-8中，每个代码点被映射到一个、两个、三个或者四个代码单元。
* UTF-16中，值小于等于U+FFFF的代码点被映射到单个代码单元中，对于值大于U+FFFF的代码点，每个代码点需要两个代码单元。在
  UTF-16 中，这些代码单元对有一个独特的术语：“Unicode 代理对”。

综上，我们可以知道`Utf16CharacterStream`中每一个UTF-16代码单元的含义：

    UTF-16中，一个代码单元要么是一个16位的代码点，要么是“Unicode代码对”中的一半，其中“unicode代码对”是一个21位的代码点。

简而言之，`Utf16CharacterStream`中的每个代码单元，要么是一个独立的unicode字符，要么是一个unicode字符的一半（和下一个代码单元组合起来才能形成一个unicode字符）。

v8实际上就是基于utf16对输入字符序列进行词法分析的！这也是Ecmascript规范所要求的：

    ECMAScript 源代码文本使用 Unicode 3.0 或更高版本的字符编码的字符序列来表示。
    符合 ECMAScript 的实现不要求对文本执行正规化，也不要求将其表现为像执行了正规化一样。
    本规范的目的是假定 ECMAScript 源代码文本都是由16位代码单元组成的序列。
    像这样包含16位代码单元序列的源文本可能不是有效的 UTF-16 字符编码。
    如果实际源代码文本没有用16位代码单元形式的编码，那就必须把它看作已经转换为 UTF-16 一样处理。

v8词法分析首先要做的是把输入字符序列转换成utf16 stream。

那如何把utf8编码的js文件转换成utf16 stream呢？

<!-- more -->

## 字节流

首先，我们需要先获得utf8字节流：

```cpp
// 二进制方式打开
FILE* file = fopen(filename, "rb"); 
// 移到文件尾
fseek(file, 0, SEEK_END);
// 字节数
int file_size = ftell(file);
// 回到文件头
rewind(file);
// 字节数组
byte* chars = new byte[file_size]; // typedef unsigned char byte
fread(chars, 1, file_size, file);
chars[file_size] = 0
```

假设文件内容如下：

```javascript
function 招聘() {
    var 信息 = "复合搜索部前端工程师";
}
```

utf8字节流如下：

```
66 75 6E 63 74 69 6F 6E 20 E6 8B 9B E8 81 98 28 29 20 7B 0A 20 20 20 20 76 61 72 20 E4 BF A1 E6 81 AF 20 3D 20 22 E5 A4
8D E5 90 88 E6 90 9C E7 B4 A2 E9 83 A8 E5 89 8D E7 AB AF E5 B7 A5 E7 A8 8B E5 B8 88 22 3B 0A 7D 0A
```

## 转换

现在，我们需要把utf8的字节流转换成utf16字符流，前面我们说了，词法分析器扫描的是`Utf16CharacterStream`类，不过这个类是一个抽象类，不能直接使用，
v8用了一个`BufferedUtf16CharacterStream`类来提供具体的功能，它继承至`Utf16CharacterStream`：

{% asset_img 1.png %}

这个`BufferedUtf16CharacterStream`提供了一个缓冲区`buffer_`，我们转换的时候，就是要向这个缓冲区填充字符。

具体的转换功能由一个叫`Utf8ToUtf16CharacterStream`的类提供，它继承至`BufferedUtf16CharacterStream`：

{% asset_img 2.png %}

基本用法：

```cpp
// utf8字节流数组头部指针，指向第一个字节
unsigned char* source_;
// 流指针
BufferedUtf16CharacterStream* stream_ = new Utf8ToUtf16CharacterStream(source_, length);
```

在生成一个`BufferedUtf16CharacterStream`实例之时，在其构造函数中，会进行第一次字符填充。

在`Utf8ToUtf16CharacterStream::FillBuffer`方法中，有这么一段代码：

```cpp
while (i < kBufferSize - 1) {
    if (raw_data_pos_ == raw_data_length_) break;
    // 取出当前字节
    unibrow::uchar c = raw_data_[raw_data_pos_];
    if (c <= unibrow::Utf8::kMaxOneByteChar) {
        // 单字节字符
        raw_data_pos_++;
    } else {
        // 多字节字符，向前继续读字节，计算出字符
        c =  unibrow::Utf8::CalculateValue(raw_data_ + raw_data_pos_,
                raw_data_length_ - raw_data_pos_,
                &raw_data_pos_);
    }
    if (c > kMaxUtf16Character) {
        // 如果计算出来的字符超出U+FFFF，那就需要两个16位代码单元来表示，也就是我们前文所说的“Unicode代码对”
        buffer_[i++] = unibrow::Utf16::LeadSurrogate(c);
        buffer_[i++] = unibrow::Utf16::TrailSurrogate(c);
    } else {
        // 如果计算出来的字符小于U+FFFF，那就直接填入buffer_
        buffer_[i++] = static_cast<uc16>(c);
    }
}
```

最后的结果就是，`BufferedUtf16CharacterStream::buffer_`中的每一个元素都是一个16位代码单元，它要么是一个字符，要么是一个“Unicode代码对”的一半。

我们以上文得到的字节流为例，最后`BufferedUtf16CharacterStream::buffer_`中会类似这样：

{% asset_img 3.png %}

得到所有字符之后，接下来的工作就是扫描每个字符，分析出`Token`。

## 扫描

v8扫描`BufferedUtf16CharacterStream::buffer_`中的每个代码单元，然后按照Ecmascript的文法产生式分析出所有Token。

我们先来看看`Scanner::Next`这个方法：

```cpp
// 是否ASCII字符
if (static_cast<unsigned>(c0_) <= 0x7f) {
    // 从`one_char_tokens`中取出该Token
    Token::Value token = static_cast<Token::Value>(one_char_tokens[c0_]);
    // 如果取出来的Token不是ILLEGAL的话，返回该Token
    if (token != Token::ILLEGAL) {
        int pos = source_pos();
        next_.token = token;
        next_.location.beg_pos = pos;
        next_.location.end_pos = pos + 1;
        Advance();
        return current_.token;
    }
}
// 否则继续扫描
Scan();
return current_.token;
```

我们注意到上面代码中的`one_char_tokens`，这货是啥？它其实就是个大小为128的数组，其中元素大部分是Token::ILLEGAL，只有几个有效的Token：

* Token::LPAREN "("
* Token::RPAREN ")"
* Token::COMMA ","
* Token::COLON ":"
* Token::SEMICOLON ";"
* Token::CONDITIONAL "?"
* Token::LBRACK "["
* Token::RBRACK "]"
* Token::LBRACE "{"
* Token::RBRACE "}"
* Token::BIT_NOT "~"

如果没匹配到`one_char_tokens`，则调用`Scanner::Scan`继续扫描:

```cpp
switch(c0_) {
    // 匹配各种常见的字符...
    case ' '
    case '\t'
    case '='
    case '~'
    ...
    default:
        // 匹配identifier或者keyword等
        if (unicode_cache_->IsIdentifierStart(c0_)) {
          token = ScanIdentifierOrKeyword();
        } else if (IsDecimalDigit(c0_)) {
          token = ScanNumber(false);
        } else if (SkipWhiteSpace()) {
          token = Token::WHITESPACE;
        } else if (c0_ < 0) {
          token = Token::EOS;
        } else {
          token = Select(Token::ILLEGAL);
        }
        break;
}
```

注意到上面的`ScanIdentifierOrKeyword`方法，这个方法是Token分析的核心，顾名思义，它就是用来分析`identifier`或者`keyword`的：

```cpp
// 字符链
// 比如`name`这样一个Token，有4个字符，这4个字符就连成了一个LiteralScope
// 当扫描完这4个字符后，会调用`LiteralScope::Complete`，这样就完成了一个Token的分析
LiteralScope literal(this);
// 扫描unicode转义字符
if (c0_ == '\\') {
    uc32 c = ScanIdentifierUnicodeEscape();
    // Only allow legal identifier start characters.
    if (c < 0 ||
            c == '\\' ||  // No recursive escapes.
            !unicode_cache_->IsIdentifierStart(c)) {
        return Token::ILLEGAL;
    }
    // \u0020
    AddLiteralChar(c);
    return ScanIdentifierSuffix(&literal);
}

uc32 first_char = c0_;
Advance();
AddLiteralChar(first_char);

// 向前扫描字符，直至遇到一个不是有效的identifier字符为止，比如空格、(、[等
while (unicode_cache_->IsIdentifierPart(c0_)) {
    if (c0_ != '\\') {
        uc32 next_char = c0_;
        Advance();
        AddLiteralChar(next_char);
        continue;
    }
    // Fallthrough if no longer able to complete keyword.
    return ScanIdentifierSuffix(&literal);
}

// 结束一个字符链，即已经扫描出了一个identifier或者keyword
literal.Complete();

if (next_.literal_chars->is_one_byte()) {
    Vector<const uint8_t> chars = next_.literal_chars->one_byte_literal();
    // 判断是标志符还是关键词
    // v8内部定义了一系列关键词matcher，能match到的就是关键词，否则就是标志符
    return KeywordOrIdentifierToken(chars.start(),
            chars.length(),
            harmony_scoping_,
            harmony_modules_);
}

return Token::IDENTIFIER;

```

我们以前文的`buffer_`字符数组为例，经过`Scanner`扫描后，会产生类似这样的结果：

{% asset_img 4.png %}

最后，对整个文件内容扫描完的结果大概就类似这样：

    No of tokens: 12
    =>    FUNCTION at (0, 8)
    =>  IDENTIFIER at (9, 11)
    =>      LPAREN at (11, 12)
    =>      RPAREN at (12, 13)
    =>      LBRACE at (14, 15)
    =>         VAR at (20, 23)
    =>  IDENTIFIER at (24, 26)
    =>      ASSIGN at (27, 28)
    =>      STRING at (29, 41)
    =>   SEMICOLON at (41, 42)
    =>      RBRACE at (43, 44)
    =>         EOS at (45, 45)


在utf8编码的情况下，v8整个词法分析过程，大概就是这样，当然实际过程中，还有很多细节问题需要处理，比如跳过js注释、html注释
、扫描十六进制字符、8进制字符、unicode转义字符等等

这就是本人粗略看了v8词法分析源码之后的一点理解，欢迎各路大神点评！
