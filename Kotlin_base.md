# Kotlin基础知识

国际惯例Hello World

```kotlin
fun main(args : Array<String>){
    println("Hello World")
}
```

此段代码相当于Java中的

```java
public static void main(String[] args){
    System.out.println("Hello World");
}
```

* 关键字*fun*用来声明一个函数
* 参数类型写在名称的后面；变量的声明也是如此
* 可以省略代码结尾的分号

## 函数

- - -
 ```kotlin
 fun max(a: Int, b: Int): Int {
     return if ( a > b) a else b
 }

 //也可以是这样的表达式函数体
fun max(a: Int, b: Int): Int = if ( a > b) a else b

//还可以省略返回值类型
fun max(a: Int, b: Int) = if ( a > b) a else b
 ```
对于表达式体函数来说，编译器会分析作为函数体的表达式，并把它的类型作为函数的返回类型，这种分析通常被称为**类型推导**

 >在Kotlin中，if是表达式，而不是语句。语句和表达式的区别在于，表达式有值，并且能作为另一个表达式的一部分使用；而语句总是包围着它的代码块中的顶层元素，并且没有自己的值

## 变量

 - - -

```kotlin
//省略类型声明
val i = 10
val s = "hello"
var x = 100
//显示声明类型
val y: Int = 90
```

声明变量的关键字有两个：

* *val* (来自value)不可变引用。使用val声明的变量不能在初始化之后再次赋值。相当于Java中的*final*。
* *var* (来自varibale)可变引用。这种变量的值可以被改变。对应Java的普通变量。

```kotlin
val s = "this is a string"
s = 100                  //类型转换错误
val s1 = "a string"
s = "another string"    //不能重复赋值
```

>默认情况下，应该尽可能地使用*val*关键字来声明所有的Kotlin变量，仅在必要的时候换成*var*

## 字符串模版

- - -

和许多脚本语言一样，Kotlin让你可以在字符串字面值中引用局部变量，只需在变量名称前面加上字符'$'

```kotlin
fun main(args: Array<String>){
    val name = if ( args.size > 0) args[0] else "Kotlin"
    println("Hello, $name!")
}
//也可以引用更复杂的表达式，只需用花括号括起来
fun main(args: Array<String>){
    if ( ags.size > 0) {
        println("Hello, ${args[0]!}")
        println("Hello, ${ if (args.size > 0) args[0] else "someone"}!")
    }
}
```

## 类和属性

- - -

```java
public class Person{
    private final String name;
    public Person(String name){
        this.name = name;
    }
    public String getName(){
        return name;
    }
}
```

在Java中，构造方法的方法体中常常包含完全重复的代码：把参数赋值给有着相同名称的字段。而在Kotlin中，可以简化成

```kotlin
class Person(val name: String)
```

这种类（只有数据没有其他代码）通常被叫作**值对象**

```kotlin
class Person(
    val name: String          //只读属性：生成一个字段和简单的getter
    var isMarried: Boolean   //可写属性：一个字段、一个getter和一个setter
)
```

基本上，当你声明属性的时候，你就声明了对应的访问器（只读属性有一个getter，可写属性有getter和setter）。如果有需要，可以使用自定义的访问器。

```kotlin
//自定义访问器
class Rectangle(val height: Int, val width: Int){
    val isSquare: Boolean               //声明属性的getter
        get(){
            return height == width
        }
}
```

## when结构

- - -

### 1. 使用"when"处理枚举类

```kotlin
//声明一个枚举类
enum class Color{
    RED, GREEN, BLACK,BLUE
}
//声明一个带属性的枚举类
enum class Color(var r: Int, var g: Int, var b: Int){
    RED(255,0,0), GREEN(0,255,0),BLACK(0,0,0),BLUE(0,0,255);   //此处一定要有分号
    fun rgb() = (r * 256 + g) * 256 + b
}
```

>在Kotlin中唯一一处需要**使用分号**的地方：如果在枚举类中定义任何方法，就要使用分号把枚举常量列表和方法定义分开。

```kotlin
fun getMnemonic(color: Color) =
    when(color){
        Color.RED ->  "red"
        Color.GREEN -> "green"
        Color.BLACK -> "black"
        Color.BLUE -> "blue"
    }
```

和Java不一样，不需要在每个分支上写break语句。如果匹配成功，只有对应的分支会执行。也可以把多个值合并到一个分支，只需要用逗号隔开这些值。

```kotlin
//导入枚举常量后不用限定词就可以访问
import com.ex.Color
import com.ex.Color.*
fun getWarmth(color: Color) =
    when(color){
        RED,GREEN -> "1"
        BLACK,BLUE -> "2"
    }
```

### 2. 在"when"结构中使用任意对象

```kotlin
fun mix(c1: Color, c2: Color) =
    when(setOf(c1, c2)){
        setOf(RED,BLACK) -> "1"    //创建一个set
        setOf(GREEN,BLUE) -> "2"
        else -> "default"
    }
```

set集合中的顺序不重要，只要两个set中包含一样的条目，它们就是相等的。假如没有分支匹配，则else分支会执行，相当与Java中的default

### 3. 使用不带参数的"when"

```kotlin
fun mixOptimized(c1: Color, c2: Color) =
    when{       //没有实参传给"when"，结果同上
        (c1 == RED && c2 == BLACK) || (c1 == BLACK && c2 == RED) -> "1"
        (c1 == GREEN && c2 == BLUE) || (c1 == BLUE && c2 == GREEN) -> "2"
        else -> "default"
    }
```

如果没有给when表达式提供参数，分支条件就是任意的布尔表达式。

## 智能转换：合并类型检查和转换

- - -

```kotlin
//表达式类层次结构
interface Expr
class Num(val value: Int) : Expr            //表示实现Expr接口
class Sum(val left: Expr, val right: Expr) //Sum运算的实参可以是任何Expr:Num或者Sum

fun eval(e: Expr) : Int{
    if(e is Num){
        val n = e as Num    //显示地转换成类型Num是多余的
        return n.value
    }
    if(e is Sum){           //变量e被智能地转换了类型
        return eval(e.right) + eval(e.left)
    }
    throw IllegalArgumentException("Unknown expression")
}
```

在Kotlin中，使用**is**判断一个变量是否是某种类型，一旦判断成功，后面就不需要转换它，这称之为**智能转换**
智能转换只在变量经过is检查且之后不再发生变化的情况下有效。在这里，这个属性必须是一个val属性，而且不能有自定义的访问器。否则，每次对属性的访问是否都能返回同样的值将无从验证。

```kotlin
fun eval(e: Expr) : Int =
    if(e is Num){
        e.value
    }else if(e is Sum){
        eval(e.right) + eval(e.left)
    }else{
        throw IllegalArgumentException("Unknown expression")
    }
```

如果if分支中只有一个表达式，花括号是可以省略的。如果if分支是一个代码块，代码块中的最后一个表达式会被作为结果返回。
进一步改写代码

```kotlin
fun eval(e: Expr) : Int =
    when(e){
        is Num -> e.value
        is Sum -> eval(e.right) + eval(e.left)
        else -> throw IllegalArgumentException("Unknown expression")
    }
```

## 迭代

- - -

Kotlin中`while`循环与Java一样，`for`循环类似`for-each`。用"`..`"表示**区间**的概念。

```kotlin

fun main(args: Array<String>){
    //打印1到100，注意..区间是闭合的，即100也会取到
    for(i in 1..100){
        println(i)
    }
    //打印100到1的偶数
    for(i in 100 downTo 1 step 2){
        println(i)
    }

    //使用until，半闭合区间，此处不包含100
    for(i in 0 until 100){
        println(i)
    }
}
```

### 使用`in`检查集合和区间成员

使用`in`运算符检查一个值是否在区间中，或者它的逆运算`!in`，来检查这个值是否不在区间中。

```kotlin
fun isLetter(c: Char) = c in 'a'..'z' || c in 'A'..'Z'
fun isNotDigit(c: Char) = c !in '0'..'9'

//用in检查作为when的分支
fun recognize(c: Char) = when(c){
    in '0'..'9' -> "It's a digit!"
    in 'a'..'z', in 'A'..'Z' -> "It's a letter!"
    else -> "I don't known..."
}
```

## Kotlin中的异常

- - -

Kotlin的异常处理和Java以及其他许多语言的处理方式相似，一个函数可以正常结束，也可以在出现错误的情况下抛出异常。方法的调用者能捕获到这个异常并处理它；如果没有被处理，异常会沿着调用栈子再次抛出。

```kotlin
fun readNumber(reader: BufferedReader): Int? {   //不必显式地指定这个函数肯能抛出的异常
    try{
        val line = reader.readLine()
        return Integer.parseInt(line)
    }catch(e: NumberFormatException){             //异常类型再右边
        return null
    }finally{                                   //finally的作用和Java中的一样
        reader.close()
    }
}
```

当然，也可以用"`try`"作为表达式

```kotlin
fun readNumber(reader: BufferedReader){
    val number = try{
        Integer.parseInt(reader.readLine())
    }catch(e: NumberFormatException){
        return
    }
    println(number)
}
```

Kotlin中的`try`关键字就像`if`和`when`一样，引入了一个表达式，可以把它的值赋给一个变量。不同于`if`，你总是需要用花括号把语句主体括起来。
如果一个`try`代码块执行一切正常，代码块最后一个表达式就是结果。如果捕获到了一个异常，相应`catch`代码块中最后一个表达式救赎结果。