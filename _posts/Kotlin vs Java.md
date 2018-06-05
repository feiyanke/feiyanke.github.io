---
date: 2017-07-2
layout: 'post'
tags:
    - 'kotlin'
    - 'java'
status: 'public'
---

# Kotlin vs Java

##  control flow
kotlin中大部分的控制流都是表达式（expression）而不是语句（statement），包括`if else`,`when`,等，除了循环`for`,`while`。而java中几乎所有的控制流都只是statement。它们之间的区别在于expression有一个值，并可以作为另一个expression的一部分，而statement没有值，不能包含在另一个expression或statement中，只能单独使用。

```
//java
String a;
if (...) {
    a = "1"
} else {
    a = "2"
}

//kotlin
val a = if (...) "1" else "2"
```
另外，kotlin中的赋值是statement，而java中的赋值是expression。
## string template

```
//java
int a = 1
String b = "a="+a

//kotlin
val a = 1
val b = "a=$a"
```
## variable

```
//java
int a = 1;
//kotlin
var a:Int = 1 //or
var a = 1 //kotlin可以进行类型推断

//java
final int a = 1;
//kotlin
val a =1;
```
kotlin 建议更多的使用`val`，尽可能避免过多的副作用

## property
kotlin class中没有field，只能定义property，实际上就是`getter`,`setter`方法的语法糖。
property也可以定义为`val`或`var`，但语义与variable不太一样。`val`表示read-only，只有`getter`没有`setter`，`var`表示read-write，既有`getter`也有`setter`

```
class Rectanle(val x, val y) {
    val isSquare = x==y
    val area = x*y
}
```
java中没有property

## when

kotlin中没有`switch`，取而代之的是`when`。`when`也是一个表达式，可以返回一个值，并且不用每个分支都用`break`来分开：

```
//java
int a;
switch (b) {
    case 1:a=1;break;
    case 2:a=2;break;
    default:a=-1;break;
}

//kotlin
val a = when(b) {
    1->1
    2->2
    else->-1
}
```
`when`中可以使用`in``is`表达式，这个是`java`没有的

```
var b:Any
when(b) {
    is Int->doSomething()
    is String->doSomething()
    else->doSomething()
}
when(b) {
    in 1..3->doSomething()
    in 6..10->doSomething()
    else->doSomething()
}
```
另外，`when`中还可以不带参数，如此每个分支都需要提供一个Boolean的判断条件：

```
when {
    a>0 -> doSomething()
    b>0 -> doSomething()
    else -> doSomething()
}
```
## for loop
kotlin只有`for <value> in <iterable>`一种语法，基本与java一致

```
//java
for(int i=0;i<len;i++) {
    doSomething()
}

//kotlin
for(i in 0..len) {
    doSomething()
}

//java
for(int v:list) {
    doSomething()
}

//kotlin
for (v in list) {
    doSomething()
}
```

## Range
kotlin中的Range实现名为Progression，包括`IntProgression, CharProgression, LongProgression`，与Python中的Range类似，常用于`for`迭代中，并且有专门关键字和一些的`infix`函数来创建：
```
val r1 = IntProgression.fromCloseRange(0,100,1) //[0,100],step=1
val r2 = 0..100 //等价于IntProgression.fromCloseRange(0,100,1)
val r3 = 0..100 step 2 //等价于IntProgression.fromCloseRange(0,100,2)
val r4 = 0 until 100 //等价于IntProgression.fromCloseRange(0,100-1,1)
val r5 = 100 downTo 1 step 2 //等价于IntProgression.fromCloseRange(100,0,-2)
```
还可以使用`in`来判断某个值是否在Range中：

```
if (v in 0..100) {
    doSomethin()
}
```
另外，kolin还支持`ComparableRange`，例如：`"111".."222"`，可以用`in`判断一个字符串是否在这个范围内（字符串是可比较的）。实际上任何实现了`Comparable`的类都可以生成这种Range并进行`in`的判断。这种方式简化了通常的range判断逻辑。

java中没有这种语法
## Collection Unpack
kotlin 支持集合的unpack操作：

```
val list = listOf(1,2,3)
val (a,b,c) = list
//a=1,b=2,c=3
```
另外Pair和Triple也支持unpack。
值得注意的是，这种操作只能在变量定义时使用，像下面这样是非法的（区别于Python）：

```
var a = 1
var b = 2
(a,b) = (b,a) //交换a,b的值
```
感觉上，kotlin的unpack更像只是变量初始化的一种语法糖，由于没有原生的`(..)`(python)，因此不能作为通常程序的一部分
另外，unpack也可以用于`for`中：

```
for((k,v) in map) {
    doSomething()
}
```

java也没有unpack语法

## function
1. kotlin可以定义顶层方法，java不行
2. kotlin方法定义可以设置默认参数，java不行
3. kotlin方法传递可以使用命名参数，java不行
4. kotlin方法的可变参数需要使用关键字`vararg`，java使用`...`。kotlin支持所谓的`spread`操作：`val a = listOf(1, *array1, *array2)`，这是java所不支持的


## package and import
kotlin中package不一定要与目录结构一致，并且`public`类名也不需要与文件名一致。
kotlin中import语句可以直接引用顶层方法或是类中的定义的静态成员等，作用与java的`static import`一致

另外kotlin还支持类似`python`的`as`语句，用于起别名，这样可以避免命名的冲突问题：

```
import
```
## extension
kotlin提供一种所谓`extension function/property`来支持在类定义之外对类方法或者属性进行扩展：

```
fun String.last() = this.get(this.length-1)
```
实际上这也是静态方法的语法糖，这里的`last`实际上相当于静态定义的函数`fun last(val this:String)`，这里`String`是这个方法的`reveiver type`，在方法定义中可以通过`this`来引用这个`reveiver`（如果命名没有冲突，这个`this`，也是可以省略的），看起来就好像这个方法是在`String`类中定义的。但是，只能引用类的`public`成员，其他成员也是不可见的。
另外，`extension`也不具有多态性

## infix call
之前的创建Range所用的`downTo``Step`都是infix call，看起来像关键字，实际上只是一个方法调用的特殊写法：
```
val a = 1 to "1" //create Pair(1, "1")，等价于
val a = 1.to("1") //to是一个infix extension function：
public infix fun <A, B> A.to(that: B): Pair<A, B> = Pair(this, that)
```
infix可以用于：1. 类的成员放方法或者扩展方法，2. 方法仅包含一个参数。

## interface

1. kotlin的interface可以有方法的默认实现：

## cast

1. smart cast using `is`

```
//kotlin
fun test(v:Any) : Int {
    if(v is String) {
        return v.length
    } else {
        return -1
    }
}

//java
int test(Object v) {
   if(v instanceof String) {
       String s = (String)v;
       return s.length()
   } else {
       return -1
   }
}
```
符合下面两个条件，smart cast就是可以工作的：
1). 变量在`is`判断后没有被改变
2). 类中的属性必须是`val`，并且没有自定义的`getter`

2. explicit cast using `as`

```
//kotlin
val a = o as String
val a = o as? String //if can't cast, a = null

//java
String a = (String)o
```


## Any
## Delegation
## Data Class
## inline
## lambda
## cast as
## in out




​    