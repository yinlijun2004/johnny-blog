---
title: kotlin初探
date: 2017-05-25 12:33:34
tags: [android, java, kotlin]
---

最近google推出了新的Android开发语言[kotlin](http://kotlinlang.org/)，花了点时间了解一下皮毛。

在[kotlin](http://kotlinlang.org/)官网上的资料还是比较丰富的，提供了一个在线的[编辑器](https://try.kotlinlang.org/#/Examples/Hello,%20world!/Simplest%20version/Simplest%20version.kt)，可以写一些小的程序片段。另外提供了一个代码翻译的工具，可以直接将java代码片段翻译成kotlin。

例如以下java代码：
```java
 class Greeting {
	private String greeting;
	public Greeting(String g) {
		greeting = g;
	}
	public void sayHello(String you) {
		System.out.println(greeting + " " + you);
	}
	public static void main(String[] argv) {
		Greeting greeting = new Greeting("Hello");
		greeting.sayHello(argv[0]);
	}
}
```
对应的kotlin代码如下：
<!--more-->
```kotlin
internal class Greeting(g:String) {
  private val greeting:String
  init{
    greeting = g
  }
  fun sayHello(you:String) {
    println(greeting + " " + you)
  }
  companion object {
    @JvmStatic fun main(argv:Array<String>) {
      val greeting = Greeting("Hello")
      greeting.sayHello(argv[0])
    }
  }
}
```

## 语法
kotlin语法很多特性都有javascript ES6的影子，比如字符串模板，解构赋值，箭头函数，元编程。

### 包定义。
包定义必需位于源文件的顶端，格式如下
```kotlin
package my.demo

import java.util.*

// ...
```
这个跟java是一样的。

### 函数
#### 定义两个Int类型参数的求和函数 
```kotlin 
fun sum(a: Int, b: Int): Int {
    return a + b
}
```
上面的函数可以有更简单的表达式写法
#### 定义表达式函数
可以看出表达式函数的返回值是自动推断的。
```kotlin
fun sum(a: Int, b: Int) = a + b
```
### 定义空返回值函数
```kotlin
fun printSum(a: Int, b: Int): Unit {
    println("sum of $a and $b is ${a + b}")
}
```
其中<b>Unit</b>可以省略
```kotlin
fun printSum(a: Int, b: Int): {
    println("sum of $a and $b is ${a + b}")
}
```
参见[函数](http://kotlinlang.org/docs/reference/functions.html)

### 局部变量
#### 定义常量（只允许赋值一次）
```kotlin
val a: Int = 1  // 立即赋值
val b = 2   // 类型推断，为`Int`
val c: Int  // 如果没有给出初始化的值，需要给出变量类型
c = 3       // 延迟赋值
```

#### 定义变量
```kotlin
var x = 5 // 类型推断
x += 1
```
参见[属性和域](http://kotlinlang.org/docs/reference/properties.html)


### 注释
kotlin的注释跟java是一样的，支持行和块两种注释方式：
```
// This is an end-of-line comment

/* This is a block comment
   on multiple lines. */
```
跟java不一样的是，kotlin的块注释是可以嵌套的。
```kotlin
/*
 *This is a block comment
 *  on multiple lines
 /* This is a nested block comment.*/
 *This is a block comment
 *  on multiple lines
 */
```
参见[注释](http://kotlinlang.org/docs/reference/kotlin-doc.html)

### 字符串模板
```kotlin
var a = 1
// 简单的字符串模板：
val s1 = "a is $a" 

a = 2
// 任意表达式的模板：
val s2 = "${s1.replace("is", "was")}, but now is $a"
```
参见[字符串模板](http://kotlinlang.org/docs/reference/basic-types.html#string-templates)

### 条件表达式
kotlin没有三目运算符，取而代之的是条件表达式。

例如一个常规的求最大值函数
```kotlin
fun maxOf(a: Int, b: Int): Int {
    if(a > b) {
        return a
    } else {
        return b
    }
}
```
假如用条件表达式可以这样写
```kotlin
fun maxOf(a: Int, b: Int) = if(a > b) a else b
```
参见[if表达式](http://kotlinlang.org/docs/reference/control-flow.html#if-expression)

### 使用nullable和判断null
如果一个值可能是null的话必须显示的标记出来，如
如果<b>str</b>转换成<b>Int</b>失败，返回<b>null</b>:
```kotlin
fun parseInt(str: String): Int? {
    //...
}
```
使用返回nullable的函数。
```kotlin
fun printProduct(arg1: String, arg2: String) {
    val x = parseInt(arg1)
    val y = parseInt(arg2)
    if(x != null && y != null) {
        println(x * y)
    } else {
        println("'$arg1' 或者 '$arg2' 不是数字")
    }
}
```
参见[安全的null](http://kotlinlang.org/docs/reference/null-safety.html)

### 类型检查和自动转换
这个操作用来检查一个对象的类型，局部变量或者属性经过判断之后，不用再显示的转换了，例如。

```kotlin
fun getStringLength(obj: Any): Int? {
    if(obj is String) {
        //obj在这个分支内自动转为"String"类型
        return obj.length
    }
    //在上面的类型检查分支之外，类型仍然为"Any"
    return null
}
```
另一种方式
```kotlin
fun getStringLength(obj: Any): Int? {
    if(obj !is String) {
        return null
    }
    return obj.length
}
```
甚至可以这样
```kotlin
fun getStringLength(obj: Any): Int? {
    //obj在&&右侧自动转换为String类型
    if(obj is String && obj.length > 0) {
        return obj.length
    }
    return null
}
```
参见[类](http://kotlinlang.org/docs/reference/classes.html)和[类型转换](http://kotlinlang.org/docs/reference/typecasts.html)

### for循环
```kotlin
val items = listOf("apple", "banana", "kiwi")
for(item in items) {
    println(item)
}
```
或者
```kotlin
val items = listOf("apple", "banana", "kiwi")
for(i in items.indices) {
    println("item at $i is ${item[i]}")
}
```
参见[for循环](http://kotlinlang.org/docs/reference/control-flow.html#for-loops)

### while循环
```kotlin
val items = listOf("apple", "banana", "kiwi")
var i = 0
while(i < items.size) {
    println("item at $i is ${item[i]}")
    i++
}
```
参见[while循环](http://kotlinlang.org/docs/reference/control-flow.html#while-loops)

### when表达式
when通常用来代替switch和多个if else，且功能强大的多。如
```kotlin
fun describe(obj: Any): String = 
when(obj) {
    1 -> "One"
    "Hello" -> "Greeting"
    is Long -> "Long"
    !is String -> "Not a String"
    else -> "Unknow"
}
```
参见[when表达式](http://kotlinlang.org/docs/reference/control-flow.html#when-expression)

### 使用范围表达式(<b>..</b>)
#### 用in运算符检查一个数字是否在一个范围内。
```kotlin
val x = 10
val y = 9
//这是个闭区间,1和y+1都算进去。
if(x in 1..y+1) {
    println("fits in range")
}
```
#### 检查一个数字是否超出范围
```kotlin
val list = listOf("a", "b", "c")
if(-1 !in 0..list.lastIndex) {
    println("-1 is out of range")
} 
if(list.size !is list.indices) {
    println("list size is out of valid list indices range too")
}
```
#### 遍历一个范围
```kotlin
for(x in 1..5) {
    print(x)
}
```

#### 使用步进遍历范围
```kotlin
for(x in 1..10 step 2) {
    print(x)
}
for(x in 9 downTo 0 step 3) {
    print(x)
}
```
参见[范围](http://kotlinlang.org/docs/reference/ranges.html)

### 集合
遍历一个集合
```kotlin
for (item in items) {
    println(item)
}
```

使用in操作判断集合是否含有某元素，如
```kotlin
fun main(args: Array<String>) {
    val items = setOf("apple", "banana", "kiwi")
    when {
        "orange" in items -> println("juicy")
        "apple" in items -> println("apple is fine too")
    }
}
```
上述代码输出
```
apple is fine too
```

使用lambda表达式过滤map集合
```kotlin
fun main(args: Array<String>) {
    val fruits = listOf("banana", "avocado", "apple", "kiwi")
    fruits
    .filter { it.startsWith("a") }
    .sortedBy { it }
    .map { it.toUpperCase() }
    .forEach { println(it) }
}
```
上述代码输出
```
APPLE
AVOCADO
```
参见[高阶函数和lambda](http://kotlinlang.org/docs/reference/lambdas.html)

## 习题
#### 数组求和
```kotlin
/*
 * Your task is to implement the sum() function so that it computes the sum of
 * all elements in the given array a.
 */
package sum

fun sum(a: IntArray): Int {
    var s = 0
    for(n in a) {
        s += n
    }
    return s
}
```

#### 求数组最大值索引
```kotlin
/*
 * Your task is to implement the indexOfMax() function so that it returns
 * the index of the largest element in the array, or null if the array is empty.
 */
package maxindex

fun indexOfMax(a: IntArray): Int? {
	if(a.size == 0) {
        return null
    }
    var m = Integer.MIN_VALUE
    var ret = -1
    for(i in a.indices) {
        if(a[i] >= m) {
            m = a[i]
            ret = i;
        }
    } 
    return ret
}

```

#### 这是求个什么？自己看解释吧
```kotlin
/*
 * Any array may be viewed as a number of "runs" of equal numbers.
 * For example, the following array has two runs:
 *   1, 1, 1, 2, 2
 * Three 1's in a row form the first run, and two 2's form the second.
 * This array has two runs of length one:
 *   3, 4
 * And this one has five runs:
 *   1, 0, 1, 1, 1, 2, 0, 0, 0, 0, 0, 0, 0
 * Your task is to implement the runs() function so that it returns the number
 * of runs in the given array.
 */
package runs

fun runs(a: IntArray): Int {
    if(a.size == 0) {
        return 0
    }
    var ret = 1
    var cur = a[0]
    for(n in a) {
    	if(n != cur) {
            ++ret
            cur = n
        }
    }
    return ret
}
```

#### 求是否是回文字符串
```kotlin
/*
 * Your task is to implement a palindrome test.
 *
 * A string is called a palindrome when it reads the same way left-to-right
 * and right-to-left.
 *
 * See http://en.wikipedia.org/wiki/Palindrome
 */
package palindrome

fun isPalindrome(s: String): Boolean {
    if(s.length < 2) {
        return true
    }
    var len = s.length / 2
    for(i in 0..len - 1) {
        if(s[i] != s[s.length - i - 1]) {
            return false
        }
    }
    return true
}
```

#### 求落单数字
```kotlin
/*
 * Think of a perfect world where everybody has a soulmate.
 * Now, the real world is imperfect: there is exactly one number in the array
 * that does not have a pair. A pair is an element with the same value.
 * For example in this array:
 *   1, 2, 1, 2
 * every number has a pair, but in this one:
 *   1, 1, 1
 * one of the ones is lonely.
 *
 * Your task is to implement the findPairless() function so that it finds the
 * lonely number and returns it.
 *
 * A hint: there's a solution that looks at each element only once and uses no
 * data structures like collections or trees.
 */
package pairless

fun findPairless(a: IntArray): Int {
    // Write your solution here
    var map = hashMapOf<Int, Int>();
    for(n in a) {
        var i = map.get(n)
        
        map.put(n, if(i == null) 1 else i + 1)
    }
    
    for((k, v) in map) {
        if(v % 2 == 1) {
            return k
        }
    }
    return 0
}

```