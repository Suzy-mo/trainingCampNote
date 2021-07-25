# 第一行代码第三版Kotlin

## 第二章Kotlin的使用

### 1、运行

运行环境

* IntelliJ IDEA （JetBrains官方开发工具）
* 在线运行 https://try.kotlinlang.org
* AS内运行 编一个main函数

运行举例

```kotlin
fun main(){
    println("Hello Kotlin!")
}
```

在左边有个小箭头，点击后选择第一个run即可运行

### 2、与java编写的不同

* 结尾无分号

### 3、变量与函数

#### 声明变量

隐式声明（不用声明类型，kotlin有出色的类型推导机制）

```kotlin
val 变量名 //对应final变量（value）
var 变量名 //对应非final变量（variable）
```

显示声明

```kotlin
val a: Int = 10
//Int Long Short Float Double Boolean Char Byte
```

小技巧：优先使用val声明，满足不了才用var（java也要养成良好习惯，加上final）

#### 自定义函数

```kotlin
fun methodName(param1: Int, param2: Char): Int{
    return 0
}
```

函数名 参数 返回值类型

语法糖的使用：简化代码

```kotlin
fun largerNumber(num1: Int,num2: Int) = max(num1, num2)
```

可以不用写函数体+类型推到机制

#### 函数修饰符

java默认default；kotlin 默认public

|           | java                         | kotlin         |
| --------- | ---------------------------- | -------------- |
| public    | 所有类可见                   | 所有类可见     |
| privite   | 当前类                       | 当前类         |
| protected | 当前类、子类、同包路径下的类 | 当前类、子类   |
| default   | 同包路径下的类               | 无             |
| internal  | 无                           | 同一模块中的类 |

### 4、程序的逻辑控制

#### 条件语句

* if 与java的使用并无区别

* if额外功能：返回值是if语句每个条件中最后一行代码的返回值

  ```kotlin
  fun largerNumber(num1: Int,num2: Int) = if(num1 > num2) num1 else num2
  ```

* when(比switch更强大)

  ```kotlin
  匹配值 -> {执行逻辑}
  fun getScore(name: String) = when(name){
      "Tom" -> 86
      "jim" -> 77
      else -> 0
  }
  
  fun getScore(name: String) = when{
      name == "Tom" -> 86
      name == "jim" -> 77
      else -> 0
  }
  ```

  判断类型：关键is

  ```kotlin
  fun main(){
      val num = 10L
      checkNumber(num)
  }
  
  fun checkNumber(num: Number){
      when(num){
          is Int -> println("int")
          is Double -> println("double")
          else -> println("error")
      }
  }
  ```

  已开头的判断

  ```kotlin
  fun getScore(name: String) = when{
      name.startsWith("Tom") -> 86
      name == "jim" -> 77
      else -> 0
  }
  ```

#### 循环语句

* while语句无差别

* java的for-each加强后变成了for-in循环(for-i 舍弃)

  ```kotlin
  val range = 0..10//[1,10]
  val range = 0 until 10//[0,10)
  
  step 数字//跳过几步
  downTo//降序（默认升序）
  ```

  ```kotlin
  for(i in 0..10){}
  for(i in 0 util 10 step 2){}
  for(i in 1 downTo 10){}
  ```

### 5、面向对象编程

#### 类与对象

```kotlin
class ClassName{
	var
    var
    
    fun 
}
```

实例化(不用new)

```kotlin
val p = Person()
```

#### 继承和构造函数

可继承的类 前面加上open

```kotlin
open class person{
    
}

class Student : Person(){
    
}
```

>括号的理解

（1)、主构造函数

* 每个类都会有一个无参的主构造函数

* 也可以显示指明参数，主函数的特点是没有函数体，直接定义在类后面

  ```kotlin
  class Student(val son: String, val grade: Int) : Person(){
  }
  ```

* 子类的主构造函数调用父类中哪个构造函数，在继承的时候通过括号来指定

  ```kotlin
  open class Person(val name: String, val age：Int){}
  
  class Student(val son: String, grade: Int,name: String, val age：Int): Person(name, age){}
  
  val student = Student("a12", 5, "jack", 19)
  ```

（2)、次构造函数(其实几乎用不到)

* 当一个类主次构造器都有时，所有次必须调用主（包括间接调用）

  ```kotlin
  class Student(val son: String, grade: Int,name: String, val age：Int): Person(name, age){
      constructor(name: String, age: Int): this("", 0, name, age){
      }
      constructor(): this("", 0){
      }
  }
  
  //有三种方式
  val student1 = student()
  val student2 = student("jack", 19)
  val student3 = student("a12", 5, "jack", 19)
  
  ```

* 只有次构造函数

  ```kotlin
  class Student : Person{
      constructor(name: String, age: Int): super(name, age){
      }
  }
  ```

#### 接口

定义同java

继承

```kotlin
interface Study{
    fun Book()
}
class Student(name: String, age: Int) : Person(name, age),Study{
    override fun rook()
}

for main(){
    val student = Student("Jack",19)
    doStudy(student)
}

fun doStudy(study: Study){
    study.rook()
}
```

额外：运行接口中定义的函数进行默认实现

```kotlin
interface Study{
    fun readBook()
    fun write(){
        println("do writing")
    }
}
```

#### 数据类

架构模式中的M

通常需要重写equals().hashCode().toString()方法

```kotlin
data class Cellphone(val brand: String, val price: Double)
```

data关键字 帮助自动重写

#### 单例类

单例模式：某个类在全局最多只有一个实例，避免创建重复的对象

```kotlin
object Singleton{
    fun tset(){
	}
}

Singleton.test()
```

### 6、Lambda编程

#### 集合创建与遍历

list和set

```kotlin
//不可变
val list = listOf("1","2")
val set = setOf("1","2")
//可变
val list = mutableListOf("1","2")
list.add("3")
for(num in list){
    println(num)
}
val set = mutabSetOf("1","2")
list.add("3")
for(num in set){
    println(num)
}
//set中不可以放重复的元素
```

map

```kotlin
val map = HashMap<String, Int>()
//不建议用put和get操作
map["A"] = 1
val number = map["A"]
```

最简写法

```kotlin
val map = mapOf("A" to 1,"B" to 2)
for((word,number) in  map){
    println(word + number)
}
```

#### 集合的函数式API

lambda表达式

```kotlin
{参数名1: 类型,参数名2: 类型 -> 函数体}
```

简化过程理解

```kotlin
val list = listOf("1","2")
val lambda = { fruit: String -> fruit.length}
val maxLengthFruit = list.maxBy(lambda)

val maxLengthFruit = list.maxBy({ fruit: String -> fruit.length})
//lambda参数是最后一个参数可以放外面，唯一的话可以把括号也省了
val maxLengthFruit = list.maxBy{ fruit: String -> fruit.length}
//类型也可省 
val maxLengthFruit = list.maxBy{ fruit -> fruit.length}
//唯一参数 用it
val maxLengthFruit = list.maxBy{ it.length}
```

map函数

将集合中每个元素都映射成一个另外的值，映射规则在lambda表达式里指定，最后生成一个新的集合

```kotlin
val list = listOf("1","2")
val newList = list.map{ it.toUpperCase()}//转成大写
```

filter函数

可以结合使用  注意使用顺序，先映射会慢很多！

```kotlin
val newList = list.filter{ it.length <= 5}
				  .map{ it.toUpperCase()}//转成大写
```

any和all函数

any 至少一个  all全部满足

```kotlin
val anyResult = list.any{ it.length <= 5}
val allResult = list.all{ it.length <= 5}
```

#### Java函数式API的使用

在Koltlin代码中调用java方法，且该方法接收一个java单抽象方法接口参数（只有一个待实现的方法）

匿名类写法 舍去new，改用object

```kotlin
Thread{
    //逻辑
}.start()

button.setOnClickListener{
    
}
```

### 7、空指针检查

（1）可空类型系统

Kotlin将空指针的异常检查提前到了编译阶段

如果允许为空的话就加一个问号

```kotlin
fun doStudy(study: Study?)
```

（2）判空辅助工具

**?.**

不空的时候继续，为空则什么都不做

```kotlin
fun doStudy(study: Study?){
	study?.readBook()
    study?.writing()
}
```

**?:**

不为空则返回左边，为空返回右边

```kotlin
val c = a ?: b

//结合使用
fun getTextLength(text: String?) = text?.length ?:0
```

**!!**

非空断言工具 强行通过编译(并不是最优选择，可能会因为过于自信造成空指针)

```kotlin
val upperCase = content!!.toUpperCase()
```

let函数（可以处理1全局变量的判空问题）

```kotlin
obj.let{obj2 ->
    //具体逻辑
}
fun doStudy(study: Study?){
	study?.let{stu->
        it.readBook()
        it.writing()       
    }
}
```

study为空，什么都不做；不为空则进入lambda表达式

### 8、小技巧

#### 字符串内嵌表达式

```kotlin
"hello, ${object.name}. nice to meet you"
//唯一变量括号可省
"hello, $name. nice to meet you"
"Cellphone(brand=$brand, price=$price"
```

#### 函数的默认参数值

```kotlin
fun printParams(num: Int = 100;str: String){
    
}
//给没有默认赋值的参数传参 不需按顺序，与形参名对应传参即可
printParams(str = "world")

//默认设置好参数，不需要再写几个构造函数
class Student(val son: String = "", val grade: Int = 0,name: String = "",age：Int = 0): Person(name, age){
}
```

## 第三章 标准函数和静态方法

### 标准函数wtih run apply

简化代码 同一个对象的就不需要再写实例，直接用里面的方法

with

```Kotlin
val result = with(obj){
    //obj的上下文
    "value"//最后一行是返回值
}
```

```Kotlin
//val bulider = StringBuilder
val list = listOf("1","2","3")
val result = with(StringBuilder()){
    //obj的上下文
    append("Start\n")
    for (num in list){
        append(num).append("\n")
    }
    append("end\n")
    toString()//最后一行是返回值
}
println(result)
```

run

```kotlin
val result = StringBuilder().run{
    //obj的上下文
    append("Start\n")
    for (num in list){
        append(num).append("\n")
    }
    append("end\n")
    toString()//最后一行是返回值
}
println(result)
```

apply

```Kotlin
val list = listOf("1","2","3")
val result = StringBuilder().apply{
    //obj的上下文
    append("Start\n")
    for (num in list){
        append(num).append("\n")
    }
    append("end\n")
}
println(result.toString())
```

#### 静态方法

静态方法 类方法  不需要创建实例就能调用的；适用于工具类

在K中推荐用单例类来实现 

* 全部方法都是的

```kotlin
bject ActivityCollector {

    fun addActivity(){
        //逻辑
    }

    fun removeActivity(){
        //逻辑
    }
}
```

* 某个方法是

```kotlin
class Util{

    fun removeActivity(){
        //逻辑
    }
    
    companion object ActivityCollector {

        fun addActivity(){
            //逻辑
        }
    }
}
```

* 真正的静态方法  用注释

   @JvmStatic 只能用在单例类或者companion object上

```java
class Util{

    fun removeActivity(){
        //逻辑
    }
    @JvmStatic
    companion object ActivityCollector {

        fun addActivity(){
            //逻辑
        }
    }
}
```

* 顶层方法

K编译器会把所有顶层方法全部编译为静态方法  在任何位置都可以直接调用

创建  new->Kotlin->File/Class   要选File  输入文件名 Helper

```kotlin
fun doSomething(){
    //逻辑
}
```

在java文件中用

```java
HelperKt.doSomething()
```

