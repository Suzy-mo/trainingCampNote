# JAVA查漏补缺之基础知识

## 一、Java基础框架

* 用户交互Scanner
* 顺序结构
* 选择结构
  * if
  * Switch 
* 循环结构
  * While
  * DoWhile
  * For
  * 增强for循环
  * break continue
* 变量
* 基本数据类型
* 运算符

## 二、Scanner

java.util.Scanner

```java
//新建对象 
Scanner scanner = new Scanner(System.in);
//以空格为分界
String str = scanner.next();  
//以回车为分解
String str = scanner.nextline();
//关闭
scanner.close();

if(scanner.hasNextInt())
    int int1 = scanner.NextInt();
Double Float 
```

## 三、增强for循环

主要用于数组和集合，简化代码

```java
for(声明语句:表达式){
    //代码句子
}
for(数据类型 新名字:原名字){
    //执行 用新名字
}
```

## 四、变量及分类

变量的本质是可操作的存储空间，不同类型的变量存储在不同的内存中

声明变量即指定对应的存储空间

变量需要初始化

final 常量 只能初始化一次

![](E:\QG移动组\QG2021暑假训练营\JAVA查漏补缺\图片\变量分类.jpg)

## 五、final

* 修饰变量 赋初值后不能变
* 修饰方法 该方法不可被子类重写，但可以被重载
* 修饰类 不能被继承