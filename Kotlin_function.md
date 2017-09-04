# 函数的定义与调用

## 在Kotlin中使用集合

```kotlin
val set = hashSetOf(1,7,33)
val list = arrayListOf(1,4,2)
val map = hashMapOf(1 to "one",2 to "two",3 to "three")

val number = list.last()    //最后一个数
val max = set.max()         //最大的数
```

Kotlin没有采用它自己的集合类，而是采用标准的Java集合类，当然也有新增的方法，可参考官方文档。

## 实现一个joinToString()函数

```kotlin
fun <T> joinToString(
    collection: Collection<T>,
    separator: String,
    prefix: String,
    postfix: String
): String{
    val result = StringBuilder(prefix)
    for((index, element) in collection.withIndex()){
        if(index > 0) result.append(separator)
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}
```

```kotlin
val list = listOf(1,2,3)
println(joinToString(list,", ","(",")"))    //(1; 2; 3)
```

这个函数可以实现自定义的打印集合，但是每次调用的时候都要传入4个参数，能不能简化一下呢？

## 命名参数&默认参数值

```kotlin
joinToString(collection,separator = " ", prefix = " ", postfix = ".")
joinToString(list,postfix = ")]",prefix = "[(",separator = " ")
```

在Kotlin中，调用函数可以更优雅。当调用一个Kotlin函数时，可以显式地标明一些参数的名称。一旦指明了一个参数的名称，为了避免混淆，它之后的所有参数都需要标明名称。

Java的另一个普遍存在的问题是，一些类的重载函数实在太多了，而在Kotlin中，你可以声明带有默认参数的函数。改写上面的`joinToString`函数。

```kotlin
fun <T> joinToString(
    collection: Collection<T>,
    separator: String = " ",            //带有默认参数
    prefix: String = "",
    postfix: String = ""
): String{
    val result = StringBuilder(prefix)
    result.append(prefix)
    for((index, element) in collection.withIndex()){
        if(index > 0) result.append(separator)
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}

val list = listOf(1,2,3)
println(joinToString(list))
println(joinToString(list,"-"))             //按照先后顺序，所以此处参数是separator
println(joinToString(list,postfix = "****"))//也可以直接指定参数（推荐）
```

## 消除静态工具类：顶层函数和属性

在Java中经常有一个类存放一些静态函数，在Kotlin中，这些函数被称为**顶层函数**。
对于顶层属性

```kotlin
const val UNIX_LINE_SEPARATOR = "\n"
//相当于Java中的
public static final String UNIX_LINE_SEPARATOR = "\n";
```

## 给别的类添加方法：扩展函数和属性

理论上来说，**扩展函数**就是一个类的成员函数，但是定义在类的外面

```kotlin
fun String.lastChar(): Char = this.get(this.length - 1)
println("Hello".lastChar())         //o
```

'*String*'是被扩展的类或者接口，称之为**接收者类型**；
用来调用这个扩展函数的那个对象称之为**接收者对象**，此处'*this*'是接收者对象的一个实例。

```kotlin
import xxx.lastChar
//import xxx.*
//使用as修改导入的类或者函数的名称
import xxx.lastChar as last
```

## 作为扩展函数的工具函数

现在可以写一个`joinToString`函数的终极版本了。

```kotlin
fun <T> Collection<T>.joinToString(
    separator: String = " ",
    prefix: String = "",
    postfix: String = ""
): String {
    val result = StringBuilder(prefix)
    for((index,element) in this.withIndex()){
        if(index > 0) result.append(separator)
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}

val list = listOf(1,2,3)
print(list.joinToString())  //1 2 3
```

## 不可重写的扩展函数

在Kotlin中不能重写扩展函数。

```kotlin
open class View{
    open fun click() = println("View clicked")
}
class Button: View(){
    override fun click() = println("Button clicked")
}
//分别扩展View Button
fun View.showOff() = println("view")
fun Button.showOff() = println("button")

val view: View = Button()
view.showOff()          //view
```

>**注意** 如果一个类的成员函数和扩展函数重名，成员函数往往会被优先使用。应当牢记：如果添加一个和扩展函数同名的成员函数，那么对应类定义的消费者将会重新编译代码，这将会改变它的意义并开始指向新的成员函数。

## 扩展属性

把上面的扩展函数变为扩展属性

```kotlin
var String.lastChar: Char
    get() =get(length-1)
```

和扩展函数一样，扩展属性也像接收者的一个普通成员属性一样，这里必须定义getter函数，因为没有默认getter的实现。同理，初始化也不可以，因为没有地方存储初始值。

```kotlin
var StringBuilder.lastChar: Char
    get() = get(length-1)
    set(value: Char){
        this.setCharAt(length-1,value)
    }

vasl sb = StringBuilder("Kotlin?")
sb.lastChar = '!'
println(sb)     //Kotlin!
```

## 可变参数：让函数支持任意数量的参数

Java中通过`...`表示可变参数，在Kotlin中，使用*vararg*修饰符，允许多个参数依次进入函数。

```kotlin
fun <T> asList(vararg ts: T): List<T>{
    val result = ArrayList<T>()
    for( t in ts){
        result.add(t)
    }
    return result
}
val list = asList(1,2,3)
```

在函数中，*vararg*修饰的参数是一个数组。如果参数本身是一个数组，又想调用使用可变参数函数，则只需在参数前加上`*`，Kotlin会对其进行解包，称之为*展开运算符*。

```kotlin
val a = arrayOf(1,2,3)
val list = asList(-1,0,*a,4)
```

## 键值对的处理：中缀调用和解构声明

```kotlin
val map = mapOf(1 to "one",7 to "seven",53 to "fifty-three")
```

这行代码中的**to**不是内置结构，而是一种特殊的函数调用，称为*中缀调用*，满足

* 必须是成员函数或者扩展函数
* 只有一个参数
* 使用**infix**关键字

```kotlin
infix fun Int.shl(x: Int): Int{
    ...
}
//call extension function using infix notation
1 shl 2
//is the same as
1.shl(2)
```

## 局部函数和扩展

```kotlin
//带重复代码的函数
class User(val id: Int, val name: String, val address: String)
fun saveUser(user: User){
    if(user.name.isEmpty()){        //重复检查的字段
        throw IllegalArgumentException("Can't save user ${user.id}: empty Name")
    }
    if(user.address.isEmpty()){     //重复检查的字段
        throw IllegalArgumentException("Can't save user ${user.id}: empty Address")
    }
    //保存user到数据库
}
```

在上面的代码中，会重复检查User的各个字段，代码结构不清晰，进行改造

```kotlin
class User(val id: Int, val name: String, val address: String)
fun saveUser(user: User){
    fun validate(user: User, value: String, fieldName: String){         //声明一个局部函数来验证字段
        if(value.isEmpty()){
            throw IllegalArgumentException("Can't save user ${user.id}: empty $fieldName")
        }
    }
    validate(user, user.name, "Name")
    validate(user, user.address. "Address")     //调用局部函数进行验证
    //保存user到数据库
}
```

这样看起来好多了，不用重复验证逻辑，如果添加了其他字段，也可以很轻松的添加验证。但是，直接传递了User对象还是有点难看。其实完全可以去掉，因为局部函数能访问所在函数中的所有参数和变量。

```kotlin
class User(val id: Int, val name: String, val address: String)
fun saveUser(user: User){
    fun validate(value: String, fieldName: String){         //不用传入User对象了
        if(value.isEmpty()){
            //可以访问外部函数的参数
            throw IllegalArgumentException("Can't save user ${user.id}: empty $fieldName")
        }
    }
    validate(user.name, "Name")
    validate(user.address. "Address")     //调用局部函数进行验证
    //保存user到数据库
```

继续改进

```kotlin
class User(val id: Int, val name: String, val address: String)
fun User.validateBeforeSave(){
    fun validate(value: String, fieldName: String){
        if(value.isEmpty()){
            throw IllegalArgumentException("Can't save user ${user.id}: empty $fieldName")
        }
        validate(name,"Name")
        validate(address."Address")
    }
}
fun saveUser(user: User){
     user.validateBeforeSave()
    //保存user到数据库
}
```

将验证代码提取到扩展函数中，使得逻辑更加清晰。