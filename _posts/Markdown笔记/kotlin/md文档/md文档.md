---
---

# 介绍
Kotlin 是一种在 Java 虚拟机上运行的静态类型编程语言，被称之为 Android 世界的Swift，由 JetBrains 设计开发并开源。
Kotlin 可以编译成Java字节码，也可以编译成 JavaScript，方便在没有 JVM 的设备上运行。
在Google I/O 2017中，Google 宣布 Kotlin 成为 Android 官方开发语言。

优势：
- 简洁: 大大减少样板代码的数量。
- 安全: 避免空指针异常等整个类的错误。
- 互操作性: 充分利用 JVM、Android 和浏览器的现有库。
- 工具友好: 可用任何 Java IDE 或者使用命令行构建。


# 基础语法

**包声明**
`package com.example.main`

**默认导入**
- otlin.*
- kotlin.annotation.*
- kotlin.collections.*
- kotlin.comparisons.*
- kotlin.io.*
- kotlin.ranges.*
- kotlin.sequences.*
- kotlin.text.*

**函数定义**
关键字 fun，参数格式为：参数 : 类型
```kotlin
fun sum(a: Int, b: Int): Int {   // Int 参数，返回值 Int
    return a + b
}
```
表达式作为函数体，返回类型自动推断：
```kotlin
fun sum(a: Int, b: Int) = a + b

public fun sum(a: Int, b: Int): Int = a + b   // public 方法则必须明确写出返回类型

```

无返回值的函数(类似Java中的void)：
```kotlin
fun printSum(a: Int, b: Int): Unit { 
    print(a + b)
}


// 如果是返回 Unit类型，则可以省略(对于public方法也是这样)：
public fun printSum(a: Int, b: Int) { 
    print(a + b)
}

```
可变长参数函数：   
函数的变长参数可以用 vararg 关键字进行标识：
```kotlin
fun vars(vararg v:Int){
    for(vt in v){
        print(vt)
    }
}

// 测试
fun main(args: Array<String>) {
    vars(1,2,3,4,5)  // 输出12345
}
```

lambda(匿名函数):
```kotlin
// 测试
fun main(args: Array<String>) {
    val sumLambda: (Int, Int) -> Int = {x,y -> x+y}
    println(sumLambda(1,2))  // 输出 3
}
```

**定义常量与变量**

可变变量定义：var 关键字
`var <标识符> : <类型> = <初始化值>`
不可变变量定义：val 关键字，只能赋值一次的变量(类似Java中final修饰的变量)
`val <标识符> : <类型> = <初始化值>`

常量与变量都可以没有初始化值,但是在引用前必须初始化
编译器支持自动类型判断,即声明时可以不指定类型,由编译器判断。

```kotlin
val a: Int = 1
val b = 1       // 系统自动推断变量类型为Int
val c: Int      // 如果不在声明时初始化则必须提供变量类型
c = 1           // 明确赋值


var x = 5        // 系统自动推断变量类型为Int
x += 1           // 变量可修改

```

**注释**
Kotlin 支持单行和多行注释
```kotlin
// 这是一个单行注释

/* 这是一个多行的
   块注释。 */
```

**字符串模板**
`$`表示一个变量名或者变量值
`$varName` 表示变量值
`${varName.fun()}` 表示变量的方法返回值:
```kotlin
var a = 1
// 模板中的简单名称：
val s1 = "a is $a" 

a = 2
// 模板中的任意表达式：
val s2 = "${s1.replace("is", "was")}, but now is $a"

```

**null检查机制**
Kotlin的空安全设计对于声明可为空的参数，在使用时要进行空判断处理，有两种处理方式：
- 字段后加!!像Java一样抛出空异常，
- 字段后加?可不做处理返回值为 null 或配合 ?: 做空判断处理
```kotlin
//类型后面加?表示可为空
var age: String? = "23" 
//抛出空指针异常
val ages = age!!.toInt()
//不做处理返回 null
val ages1 = age?.toInt()
//age为空返回-1
val ages2 = age?.toInt() ?: -1
```
当一个引用可能为 null 值时, 对应的类型声明必须明确地标记为可为 null:
```kotlin
fun parseInt(str: String): Int? {
  // ...
}

```
**类型检测及自动类型转换**
使用 is 运算符检测一个对象是否是指定类型的实例(类似于 Java 中的 instanceof 关键字):
```kotlin
if (obj is Type) {
    // 如果 obj 是 Type 类型，则可以直接使用 Type 类型的属性和方法
} else {
    // 处理其他类型情况
}
```
当 is 检测通过时，Kotlin 会自动将 obj 视为指定类型，因此在 if 语句的分支内不需要显式地进行类型转换。这被称为智能类型转换。

**区间**
区间表达式由具有操作符形式 `..` 的 rangeTo 函数辅以 `in` 和 `!in` 形成。
区间是为任何可比较类型定义的，但对于整型原生类型，它有一个优化的实现

```kotlin
for (i in 1..4) print(i) // 输出“1234”

for (i in 4..1) print(i) // 什么都不输出

if (i in 1..10) { // 等同于 1 <= i && i <= 10
    println(i)
}

// 使用 step 指定步长
for (i in 1..4 step 2) print(i) // 输出“13”

for (i in 4 downTo 1 step 2) print(i) // 输出“42”


// 使用 until 函数排除结束元素
for (i in 1 until 10) {   // i in [1, 10) 排除了 10
     println(i)
}
```

# 基本数据类型
Kotlin 的基本数值类型包括 Byte、Short、Int、Long、Float、Double 等。
不同于 Java 的是，字符不属于数值类型，是一个独立的数据类型。

**整数类型**
- Byte: 8 位，范围从 -128 到 127。
- Short: 16 位，范围从 -32,768 到 32,767。
- Int: 32 位，范围从 -2^31 到 2^31 - 1。
- Long: 64 位，范围从 -2^63 到 2^63 - 1。

**浮点数类型**
- Float: 32 位，单精度，带有 6-7 位有效数字。
- Double: 64 位，双精度，带有 15-16 位有效数字。
**字符类型**
- Char: 16 位的 Unicode 字符。
**字符串类型**
- String: 一系列字符的序列。
**布尔类型**
- Boolean: 有两个值：true 和 false。

**数组类型**
- IntArray: 存储 Int 类型的数组。
- DoubleArray: 存储 Double 类型的数组。
- Array<T>: 泛型数组，可以存储任意类型。

**字面常量**
- 十进制：123
- 长整型以大写的 L 结尾：123L
- 16 进制以 0x 开头：0x0F
- 2 进制以 0b 开头：0b00001011
>注意：8进制不支持
- Doubles 默认写法: 123.5, 123.5e10
- Floats 使用 f 或者 F 后缀：123.5f
- 可以使用下划线使数字常量更易读：`val creditCardNumber = 1234_5678_9012_3456L`

**比较两个数字**
比较两个数字可以使用标准的比较运算符，包括 ==、!=、<、>、<= 和 >=
这些运算符可以比较基本数据类型
```kotlin
fun main() {
    val a: Int = 5
    val b: Int = 10
    val c: Double = 5.0

    // 相等和不相等比较
    println("a == b: ${a == b}")   // 输出 false
    println("a != b: ${a != b}")   // 输出 true

    // 大小比较
    println("a < b: ${a < b}")     // 输出 true
    println("a > b: ${a > b}")     // 输出 false
    println("a <= b: ${a <= b}")   // 输出 true
    println("a >= b: ${a >= b}")   // 输出 false

    // 不同类型的比较
    println("a == c: ${a == c}")   // 输出 true，Kotlin 自动将 Int 转换为 Double 进行比较
    println("a < c: ${a < c}")     // 输出 false
    println("a > c: ${a > c}")     // 输出 false
}
```
1. 类型转换：在比较不同数据类型时会自动进行类型转换。
2. 相等性： == 运算符用于结构相等性比较，即值的比较， === 运算符用于引用相等性比较，即对象是否是同一个实例。
```kotlin
fun main() {
    val x: Int = 1000
    val y: Int = 1000

    println("x == y: ${x == y}")   // 输出 true，值相等
    println("x === y: ${x === y}") // 输出 false，不同的实例

    val z: Int = x
    println("x === z: ${x === z}") // 输出 true，引用相同
}
```

**类型转换**

每种数据类型可以转换为其他类型：
```kotlin
toByte(): Byte
toShort(): Short
toInt(): Int
toLong(): Long
toFloat(): Float
toDouble(): Double
toChar(): Char
```

**位操作符**
```kotlin
shl(bits) – 左移位 (Java’s <<)
shr(bits) – 右移位 (Java’s >>)
ushr(bits) – 无符号右移位 (Java’s >>>)
and(bits) – 与
or(bits) – 或
xor(bits) – 异或
inv() – 反向
```
**字符**
和 Java 不一样，Kotlin 中的 Char 不能直接和数字操作，Char 必需是单引号 ' 包含起来的。比如普通字符 '0'，'a'。
字符字面值用单引号括起来: '1'。 特殊字符可以用反斜杠转义。 支持这几个转义序列：\t、 \b、\n、\r、\'、\"、\\ 和 \$。 编码其他字符要用 Unicode 转义序列语法：'\uFF00'。

我们可以显式把字符转换为 Int 数字：

```kotlin
fun decimalDigitValue(c: Char): Int {
    if (c !in '0'..'9')
        throw IllegalArgumentException("Out of range")
    return c.toInt() - '0'.toInt() // 显式转换为数字
}
```
当需要可空引用时，像数字、字符会被装箱。装箱操作不会保留同一性。

**布尔**

布尔用 Boolean 类型表示，它有两个值：true 和 false。
若需要可空引用布尔会被装箱。
内置的布尔运算有：
```kotlin
|| – 短路逻辑或
&& – 短路逻辑与
! - 逻辑非
```

**数组**
数组用类 Array 实现，并且还有一个 size 属性及 get 和 set 方法，由于使用 [] 重载了 get 和 set 方法，所以我们可以通过下标很方便的获取或者设置数组对应位置的值。

数组的创建两种方式：一种是使用函数arrayOf()；另外一种是使用工厂函数。如下所示，我们分别是两种方式创建了两个数组：
```kotlin
fun main(args: Array<String>) {
    //[1,2,3]
    val a = arrayOf(1, 2, 3)
    //[0,2,4]
    val b = Array(3, { i -> (i * 2) })

    //读取数组内容
    println(a[0])    // 输出结果：1
    println(b[1])    // 输出结果：2
}
```

如上所述，[] 运算符代表调用成员函数 get() 和 set()。

注意: 与 Java 不同的是，Kotlin 中数组是不协变的（invariant）。

除了类Array，还有ByteArray, ShortArray, IntArray，用来表示各个类型的数组，省去了装箱操作，因此效率更高，其用法同Array一样：
```kotlin
val x: IntArray = intArrayOf(1, 2, 3)
x[0] = x[1] + x[2]**字符串**

```

**字符串**
和 Java 一样，String 是不可变的。方括号 [] 语法可以很方便的获取字符串中的某个字符，也可以通过 for 循环来遍历：
```kotlin
for (c in str) {
    println(c)
}
```
kotlin支持三个引号"""括起来的字符串，支持多行字符串
String 可以通过 trimMargin() 方法来删除多余的空白。
默认 | 用作边界前缀，但你可以选择其他字符并作为参数传入，比如 trimMargin(">")。

**字符串模板**

字符串可以包含模板表达式 ，即一些小段代码，会求值并把结果合并到字符串中。 模板表达式以美元符（$）开头，由一个简单的名字构成:

```kotlin
fun main(args: Array<String>) {
    val i = 10
    val s = "i = $i" // 求值结果为 "i = 10"
    println(s)
}
```
或者用花括号扩起来的任意表达式:
```kotlin
fun main(args: Array<String>) {
    val s = "runoob"
    val str = "$s.length is ${s.length}" // 求值结果为 "runoob.length is 6"
    println(str)
}
```

原生字符串和转义字符串内部都支持模板。 如果你需要在原生字符串中表示字面值 $ 字符（它不支持反斜杠转义），你可以用下列语法：
```kotlin
fun main(args: Array<String>) {
    val price = """
    ${'$'}9.99
    """
    println(price)  // 求值结果为 $9.99
}
```
















# 未完待续






# kotlin 扩展
Kotlin 可以对一个类的属性和方法进行扩展，且不需要继承或使用 Decorator（装饰器） 模式。
扩展是一种静态行为，对被扩展的类代码本身不会造成任何影响。

**扩展函数**
扩展函数可以在已有类中添加新的方法，不会对原类做修改，扩展函数定义形式：
```kotlin
fun receiverType.functionName(params){
body
}
```
- receiverType：表示函数的接收者，也就是函数扩展的对象
- functionName：扩展函数的名称
- params：扩展函数的参数，可以为NULL
> this关键字指代接收者对象(receiver object)(也就是调用扩展函数时, 在点号之前指定的对象实例)。

**扩展函数是静态解析的**
扩展函数是静态解析的，并不是接收者类型的虚拟成员，在调用扩展函数时，具体被调用的的是哪一个函数，由调用函数的的对象表达式来决定的，而不是动态的类型决定的:
若扩展函数和成员函数一致，则使用该函数时，会优先使用成员函数。

**扩展一个空对象**
在扩展函数内， 可以通过 this 来判断接收者是否为 NULL,这样，即使接收者为 NULL,也可以调用扩展函数。


**扩展属性**
扩展属性允许定义在类或者kotlin文件中，不允许定义在函数中。
初始化属性因为属性没有后端字段（backing field），所以不允许被初始化，只能由显式提供的 getter/setter 定义。
扩展属性只能被声明为 val

**伴生对象的扩展**
如果一个类定义有一个伴生对象 ，你也可以为伴生对象定义扩展函数和属性。
伴生对象通过"类名."形式调用伴生对象，伴生对象声明的扩展函数，通过用类名限定符来调用：
```kotlin
class MyClass {
    companion object { }  // 创建一个伴生对象（名称默认为 Companion）
}

// 为 MyClass 的伴生对象添加扩展函数
fun MyClass.Companion.foo() {
    println("伴随对象的扩展函数")
}

// 为 MyClass 的伴生对象添加扩展属性
val MyClass.Companion.no: Int
    get() = 10

fun main(args: Array<String>) {
    // 通过类名直接访问伴生对象的扩展属性
    println("no:${MyClass.no}")  // 输出: no:10
    
    // 通过类名直接调用伴生对象的扩展函数
    MyClass.foo()  // 输出: 伴随对象的扩展函数
}
```
**扩展的作用域**
扩展的可见性取决于扩展的定义位置
```kotlin
// 文件: Extensions.kt

// 1. 顶级扩展（文件作用域）
fun String.addExclamation(): String = "$this!"

class User(val name: String) {
    // 2. 类内部扩展（类作用域）
    fun String.withinClass(): String = "Within class: $this"
    
    fun test() {
        "Hello".withinClass()  // ✅ 可以访问
        "Hello".addExclamation()  // ✅ 可以访问（同一文件）
    }
}

// 3. 私有扩展（仅当前文件可见）
private fun String.privateExtension() = "Private: $this"

// 4. 内部扩展（仅当前模块可见）
internal fun String.internalExtension() = "Internal: $this"

class AnotherClass {
    fun test() {
        "Hello".addExclamation()  // ✅ 同一文件，可以访问
        "Hello".privateExtension()  // ✅ 同一文件，可以访问
        "Hello".internalExtension()  // ✅ 同一文件，可以访问
        // "Hello".withinClass()  // ❌ 不同类，无法访问
    }
}
```

**扩展声明为成员**
在一个类内部你可以为另一个类声明扩展。
在这个扩展中，有个多个隐含的接受者，其中扩展方法定义所在类的实例称为分发接受者，而扩展方法的目标类型的实例称为扩展接受者。

```kotlin
// 定义一个类 D，有一个 bar() 方法
class D {
    fun bar() { 
        println("D bar") 
    }
}

// 定义一个类 C，有一个 baz() 方法
class C {
    fun baz() { 
        println("C baz") 
    }

    /**
     * 为类 D 添加扩展函数 foo()
     * 这个扩展函数定义在类 C 的内部
     * 所以它拥有两个接收者：
     * 1. 扩展接收者 (Extension Receiver) - D 的实例
     * 2. 分发接收者 (Dispatch Receiver) - C 的实例
     */
    fun D.foo() {
        // bar() 是 D 的成员方法
        // 这里的 bar() 调用的是扩展接收者 (D 实例) 的方法
        bar()   // 相当于 this.bar()，this 是 D 的实例
        
        // baz() 是 C 的成员方法
        // 这里的 baz() 调用的是分发接收者 (C 实例) 的方法
        baz()   // 相当于 this@C.baz()，调用外部类 C 的方法
    }

    /**
     * 调用者方法，接收一个 D 类型的参数
     */
    fun caller(d: D) {
        // 调用 D 的扩展函数 foo()
        // 这个扩展函数是在 C 类中定义的
        // 所以调用时，需要同时有 C 的实例（通过 this）和 D 的实例（通过参数）
        d.foo()   // 相当于 this.d.foo()，这里的 this 是 C 的实例
    }
}

fun main(args: Array<String>) {
    // 创建 C 的实例
    val c: C = C()
    
    // 创建 D 的实例
    val d: D = D()
    
    // 调用 C 的 caller 方法，传入 D 的实例
    // 执行流程：
    // 1. 进入 C.caller(d)
    // 2. 在 caller 中调用 d.foo()
    // 3. 进入扩展函数 D.foo()，此时：
    //    - 扩展接收者：d (D 的实例)
    //    - 分发接收者：c (C 的实例)
    // 4. 执行 bar()：调用 d.bar() → 输出 "D bar"
    // 5. 执行 baz()：调用 c.baz() → 输出 "C baz"
    c.caller(d)
}
```








# kotlin 委托
委托模式是软件设计模式中的一项基本技巧。在委托模式中，有两个对象参与处理同一个请求，接受请求的对象将请求委托给另一个对象来处理。
Kotlin 直接支持委托模式，更加优雅，简洁。Kotlin 通过关键字 by 实现委托。

**类委托**

类的委托即一个类中定义的方法实际是调用另一个类的对象的方法来实现的。



# kotlin标准库内联函数

**repeat** 是 Kotlin 标准库提供的一个内联函数，用于重复执行某段代码指定次数。

```
repeat(times: Int, action: (Int) -> Unit)
```

- `times`: 重复执行的次数
- `action`: 要执行的代码块，可以接收当前索引（从 0 开始）





