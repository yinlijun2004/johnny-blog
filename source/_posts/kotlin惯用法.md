---
title: kotlin惯用法
date: 2017-05-26 10:01:03
tags: kotlin
---

kotlin提供了一些惯用法（语法糖？），简单的记录一下。

## data class
有时候需要一些类要保存数据，而不需要其他操作，当然基本操作除外，kotlin为data class提供的基本操作有：

- equals() 相当与“==”操作
- hashCode() 计算hash值
- toString() 函数
- componentsN() 函数
- copy() 拷贝

```kotlin

data class User(val name: String = "nobody", val age: Int = 0)

fun main(args: Array<String>) {
    //默认参数
    val nobody = User()
    //User(name=nobody, age=0)
    println(nobody)
    
    val johnny = User("Johnny", 29)
    //解构赋值
    val (name, age) = johnny
    //name: Johnny, age: 29
    println("name: $name, age: $age")
    
    //toString用法
    //User(name=Johnny, age=29)
    println(johnny)
    
    //hashCode用法
    //233064103
    println(johnny.hashCode())
    
    //"==" 和"==="
    //true
    println("johnny == johnny  ${johnny == johnny}")
    //true
    println("johnny === johnny  ${johnny === johnny}")
    
    //copy用法
    val youngJohnny = johnny.copy(age = 2)
    //"User(name=Johnny, age=2)"
    println(youngJohnny)
    //false
    println("johnny == youngJohnny  ${johnny == youngJohnny}")
    //false
    println("johnny === youngJohnny  ${johnny === youngJohnny}")
    //false
    println("johnny.hashCode() == yongJohnny.hashCode() ${johnny.hashCode() == youngJohnny.hashCode()}")
    
    val copyJohnny = johnny.copy()
    //true
    println("johnny == copyJohnny  ${johnny == copyJohnny}")
    //false
    println("johnny === copyJohnny  ${johnny === copyJohnny}")
    //true
    println("johnny.equals(copyJohnny)  ${johnny.equals(copyJohnny)}")
    //true
    println("johnny.hashCode() == copyJohnny.hashCode() ${johnny.hashCode() == copyJohnny.hashCode()}")
    
    val anotherJohnny = User("Johnny", 29)
    //true
    println("johnny == anotherJohnny  ${johnny == anotherJohnny}")
    //false
    println("johnny === anotherJohnny  ${johnny === anotherJohnny}")
    //true
    println("johnny.equals(anotherJohnny)  ${johnny.equals(anotherJohnny)}")
    //true
    println("johnny.hashCode() == anotherJohnny.hashCode() ${johnny.hashCode() == anotherJohnny.hashCode()}")
}
```
<!-- more -->

## 函数(包括构造函数)的默认值

如上例的
```kotlin
data class User(val name: String = "nobody", val age: Int = 0)
```
对于普通函数
```kotlin
fun foo(a: Int = 0, b: String = "") { ... }
```

有了默认的参数，就可以避免写多个重载函数
如下面的java代码
```java
public String foo(String name, int number, boolean toUpperCase) {
    return (toUpperCase ? name.toUpperCase() : name) + number;
}
public String foo(String name, int number) {
    return foo(name, number, false);
}
public String foo(String name, boolean toUpperCase) {
    return foo(name, 42, toUpperCase);
}
public String foo(String name) {
    return foo(name, 42);
}
```
可以用一个kotlin函数表示
```kotlin
fun foo(name: String, number: Int = 42, toUpperCase: Boolean = false) =
        (if (toUpperCase) name.toUpperCase() else name) + number
```

## 过滤list
```kotlin
val positives = list.filter { x => x > 0}
```
甚至可以更简洁一点
```kotlin
val positives = list.filter { it > 0 }
```

## 字符串模板
```kotlin
println("Name $name")
```

## 类型检查
```kotlin
when (x) {
    is Foo -> ...
    is Bar -> ...
    else   -> ...
}
```

## 遍历集合
```kotlin
for ((k, v) in map) {
    println("$k -> $v")
}
```

## 使用范围（Ranges）
```kotlin
for (i in 1..100) { ... }  // 闭区间包含100
for (i in 1 until 100) { ... } // 半开区间，不包含100
for (x in 2..10 step 2) { ... } 
for (x in 10 downTo 1) { ... }
if (x in 1..10) { ... }
```

## 构造只读列表
```kotlin
val list = listOf("a", "b", "c")
```

## 构造只读map
```kotlin
val map = mapOf("a" to 1, "b" to 2, "c" to 3)
```

## 访问map
```kotlin
println(map["key"])
map["key"] = value
```

## 延迟加载
```kotlin
val p: String by lazy {
    // compute the string
}
```

## 函数扩展
```kotlin
fun String.spaceToCamelCase() { ... }

"Convert this to camelcase".spaceToCamelCase()
```

## 创建单例
```kotlin
object Resource {
    var name = "Name"
}

fun main(args: Array<String>) {
	var rs = Resource;
    //Name
    println(rs.name)
    var rs2 = Resource;
    //Name
    println(rs2.name)
    rs.name = "Anthoer"
    //Another
    println(rs.name)
    //Another
    println(rs2.name)
}
```

## 判断null

### ?.
```kotlin
val files = File("Test").listFiles()

println(files?.size)
```
例如：
```kotlin
fun getList(isNull: Boolean): List<String>? =
    if(isNull) null else listOf("a", "b", "c")

fun main(args: Array<String>) {
    var list = getList(true)
    //"null"
    println(list?.size)
    
    list = getList(false)
    //"3"
    println(list?.size)
}
```
### ?. 可以执行语句块
val data = ...

data?.let {
    ... // execute this block if not null
}

### ?:
上述println语句可以改为
```kotlin
println(list?.size ?: "empty" )
```
这打印结果如下
```
empty
3
```
### ?: 后面的字符串也可以换成表达式
```kotlin
val data = ...
val email = data["email"] ?: throw IllegalStateException("Email is missing!")
```

## 返回when语句
```kotlin
fun transform(color: String): Int {
    return when (color) {
        "Red" -> 0
        "Green" -> 1
        "Blue" -> 2
        else -> throw IllegalArgumentException("Invalid color param value")
    }
}
```

## 'try/catch' 表达式
```kotlin
fun test() {
    val result = try {
        count()
    } catch (e: ArithmeticException) {
        throw IllegalStateException(e)
    }

    // result是count()返回值
}
```

## 'if'表达式
```kotlin
fun foo(param: Int) {
    val result = if (param == 1) {
        "one"
    } else if (param == 2) {
        "two"
    } else {
        "three"
    }
}
```


## 表达式函数
```kotlin
fun theAnswer() = 42
```
等于如下函数
```kotlin
fun theAnswer(): Int {
    return 42
}
```
表达式函数可以很方便的和其他惯用法结合在一起使用
```kotlin
fun transform(color: String): Int = when (color) {
    "Red" -> 0
    "Green" -> 1
    "Blue" -> 2
    else -> throw IllegalArgumentException("Invalid color param value")
}
```

## with语句（ES6不是快废除这个了？）
with语句块的函数都是对()括号内的对象的方法调用。
```kotlin
class Turtle {
    fun penDown()
    fun penUp()
    fun turn(degrees: Double)
    fun forward(pixels: Double)
}

val myTurtle = Turtle()
with(myTurtle) { //draw a 100 pix square
    penDown()
    for(i in 1..4) {
        forward(100.0)
        turn(90.0)
    }
    penUp()
}
```

## nullable的Boolean对象
```kotlin
val b: Boolean? = ...
if (b == true) {
    ...
} else {
    // `b` 为false或者null
}
```
