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

