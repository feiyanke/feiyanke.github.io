---
date: 2017-11-1
layout: 'post'
tags:
    - 'kotlin'
status: 'public'
---

# 使用递归下降法解析JSON

## JSON

**JSON** (JavaScript Object Notation) 是一种常用的数据格式，实际工程中几乎无处不在。简单来说，JSON就是一种特定的字符串格式，可以保存各种数据，对象和列表，用于数据的传输和交换。现在，几乎所有的语言都有专门的用于JSON解析和生成的库。

## JSON语法

Json的语法在[官网](http://www.json.org/)上有明确的定义。简单来说就是一些规则，使用这些规则，我们就可以将数据对象转换为JSON格式，或者将JSON字符串解析为我们想要的数据对象。这些规则大致包含：

```
<json>     ::= <object> | <array>
<value>    ::= <object> | <array> | <boolean> | <string> | <number> | <null>
<array>    ::= "[" [<value>] {"," <value>}* "]"
<object>   ::= "{" [<pair>] {"," <pair>}* "}"
<pair> ::= <string> ":" <value>
```

可以看到，这是一个递归的定义，例如 <array> 中包含了 <value> ，<value> 中又包含了 <array> 。递归下降就是按照这种递归定义的格式来编写的一种解析器，每个定义都写成一个函数，在函数中再根据定义递归的调用其他定义的解析子函数，最后完成整个结构的解析。接下来，我会自顶向下的说明这个解析器的构造过程。

这里将整个解析器封装在一个类中，整体的结构像下面这样：

```kotlin
class Parser(val json:String) {
  var at = 0
  fun next() = json[at++]
  val c : Char get() = json[at]
  fun s(n:Int) : String {
    return json.substring(at until at+n)
  }
  fun escapeSpace()
  fun parseNull()
  fun parseTrue()
  fun parseFalse()
  fun parseString()
  fun parseNumber()
  fun parseArray()
  fun parsePair()
  fun parseObject()
  fun parseValue()
  fun parseJson()
  fun error()
}
```

其中`at` 代表当前解析的字符所处的位置，`next` 用于得到当前字符并移动到下一字符，`c` 用于得到当前字符，`s` 用于得到当前位置之后n位的子字符串。`escapeSpace` 用于跳过空白字符，`error` 用来指示错误。

### Parse Json

从定义中，<json> 是一个 <object> 或者 <array>，<object> 以 **'{'** 开头，<value> 以 **'['** 开头，因此我们可以使用`switch` 结构来处理：

```kotlin
fun parse() {
  escapeSpace()
  return when(c) {
    '{' -> parseObject()
    '[' -> parseArray()
    else -> error()
  }
}
```

接下来我们只需要实现`parseObject` 以及 `parseArray` 就可以完成整个json文本的解析了。

### Parse Object

Object是一个无序的string/value对（<pair>）的集合，必须以`{` 开始，并以`}` 结束，string和value 之间以`:` 分隔，并且多个<pair>之间以`,` 分隔。

![img](/assets/img/object.gif)

代码如下：

```kotlin
fun parseObject() {
  if (c != '{') error()
  next()
  escapeSpace()
  if (c != '}') {
    loop@ while (true) {
      parsePair()
      escapeSpace()
      when (next()) {
        ',' -> continue@loop
        '}' -> break@loop
        else -> error()
      }
    }
  }
}
```

其中值得注意的是，由于 <object> 可以为空，因此需要先判断 **'{'** 的下一个非空白字符是否为 **'}'** ，然后再进行循环解析 <pair> 。

### Parse Pair

<pair> 结构很简单就是 <string> 和 <value> 之间加一个 **':'** ，代码如下：

```kotlin
fun parsePair() {
  parseString()
  escapeSpace()
  if (next() != ':') error()
  parseValue()
}
```

接下来我们需要实现 `parseArray`

### Parse Array

<array> 就是包含在 **'[]'** 中的一系列 <value> 的集合，并且 <value> 之间以逗号分隔：

![Parse String](/assets/img/array.gif)

代码如下：

```kotlin
fun parseArray() {
  if(next() != '[') error()
  escapeSpace()
  if (c != ']') {
    loop@ while (true) {
      parseValue()
      escapeSpace()
      when (next()) {
        ',' -> continue@loop
        ']' -> break@loop
        else -> error()
      }
    }
  }
}
```

可以看出 <array> 的结构和 <object> 基本一致，解析方法也基本相同。

至此，我们就基本上完成了主要的json结构的解析，其余的 <string> <number> <boolean> <null> 均为json数据的元类型，其解析函数就不在这里一一详述了。另外，上述代码仅仅描述了解析器的构造过程，实际上，我们在执行解析过程中，一般会生成解析树，用于之后的数据处理。完整的代码可以在 [koto](https://github.com/feiyanke/koto) 中找到。