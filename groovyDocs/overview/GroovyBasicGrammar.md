# Groovy数据类型

在任何编程语言中，需要使用各种变量来存储各类型的信息。变量只是保留值的存储位置，这意味着创建一个变量，保留在内存中的一些空间类存储与变量相关的值

## 内置数据类型

Groovy提供多种内置数据类型：byte、short、int、long、float、double、char、Boolean、String

Groovy中基本数据类型以及所占内存大小以及最大允许值都跟Java中一样

Groovy还允许其他类型的变量，如数组、结构和类

## 数字类

类型除了基本类型，还允许以下对象类型（也成包装类）

Byte、Short、Integer、Long、Float、Double

## Groovy变量

Groovy中变量可以通过两种方式定义

+ 使用数据类型的本地语法
+ 使用def关键字

对于变量定义。必须指明提供类型名称或是使用“def”，这是Groovy解析器需要的

```groovy
int x = 3;
def = 3;
```

## Groovy运算符

Groovy有除了Java中已有的 **算术运算符、关系运算符、逻辑运算符、位运算符、赋值运算符** 外，还有跟Kotlin中一样的 **范围运算符**

```groovy
def range = 5..10 //这只是定义了一个简单的整数范围，存储到一个局部变量范围内下限为0和上限为5
```

## Groovy循环

Groovy还提供了语句来改变逻辑中的控制流。

+ while语句

while语句首先计算条件表达式来执行，结果为真，这执行while循环中语句

+ for语句

用于遍历一组值

+ for-in

遍历一组值

## 循环控制语句

break语句和continue语句

## 条件语句

if语句、if/else语句、switch语句用法同Java一样

## groovy方法

groovy中方法，是使用返回类型或是def关键字定义的。方法可以接收任意数量的参数，定义参数时，不必显示定义类型。可以添加修饰符，如public，private和protected。默认修饰符为public

## groovy方法参数

一个方法的行为由一个或多个参数的只确定，参数名必须彼此不同

使用参数的最简单的放大类型，如下所示：

```groovy
def method(parameter1,parameter2,parameter3){
	println("$parameter1,$parameter2,$parameter3")
}
```
