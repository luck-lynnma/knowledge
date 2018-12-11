# 简介
Kotlin 是一门非常棒的现代编程语言。它最初的目标是弥补 Java 的缺点，与 Java 的可互操作性使我们可以同时使用两者混合编程，Kotlin 保持与现有基于 Java 的技术栈的完全兼容性，它可以用作服务端开发。Kotlin 也非常适合开发 Android 应用程序，将现代语言的所有优势带入 Android 平台而不会引入任何新的限制，Google 在2017年 Google I/O 大会上宣布 Kotlin 成为 Android 官方开发语言。Kotlin 提供了 JavaScript 作为目标平台的能力，支持构建基于浏览器的应用。Kotlin/Native 技术支持将Kotlin编译为原生二进制文件，目前支持Windows、Linux、MacOS、iOS、Android、WebAssembly平台，可以预见 Kotlin/Native 将会支持我们实现“一次编写、到处编译”的梦想。得益于 JetBrains，Kotlin 有很棒的 IDE 支持及很多的学习资料。对于 Java 开发人员 Kotlin 入门很容易。


本教程将引导使用 Android Studio、IntelliJ IDEA 开发工具作为 kotlin 语法学习的工具。


>如果，您想首先熟悉下Kotlin用法，以及一些好玩的功能，请参考[Kotlin快速使用](/Kotlin快速使用.md)


# Lessons

+ [Kotlin概述与常用基础语法](/overview/1.Kotlin概述与常用基础语法_v1.1.md)  

  包含Kotlin基本语法知识，像包、变量常量、函数（包括顶层函数）与属性、基本数据类型、集合、区间（range）、控制流（if，when，for，while）等相关内容

+ [Kotlin类与对象一](/overview/2.Kotlin类与对象一.md)  

  包含Kotlin关于类与对象基本语法，例如声明类，类的主/次构造函数，类的继承，继承中超类的主构造函数初始化问题，方法及属性重写，以及抽象类与接口相关知识类等内容

+ [Kotlin类与对象二](/overview/3.Kotlin类与对象二.md)  

  本次课程所包含几种类型的类，像数据类、密封类、枚举类内容的介绍

+ [Kotlin类与对象三](/overview/4.Kotlin类与对象三.md)  

  本次课程所包含的内容如下:  
   + Kotlin可见修饰符
   + Kotlin扩展
   + Kotlin委托、延迟属性、延迟加载可观察属性
   + Kotlin中对象表达式、对象声名、伴生对象

+ [Kotlin函数与Lambda表达式](/overview/5.Kotlin函数与Lambda表达式.md)  

  本节课程主要包括内容如下：  
   + 函数-包含函数的参数、用法、以及常见函数等
   + Lambda表达式、匿名函数
   + 高阶函数
   + 内联函数

+ [Kotlin基础语法补充一](/overview/6.Kotlin基础语法补充一.md)  

  本节课包含内容如下：  
   + 解构声明
   + 注解
   + This表达式
   + 相等性
   + 操作符重载

+ [Kotlin泛型](/overview/7.Kotlin泛型.md)  

  本节课主要介绍Java和Kotlin中的泛型

* [Kotlin基础语法补充二](/overview/8.Kotlin基础语法补充二.md)  

  本节课所包含的内容如下：  
   + 中缀表示法(infix)
   + 空安全
   + 异常
   + 反射
   + 类型安全的构建器
   + 类型别名
   + 多平台项目等相关内容学习


* [Kotlin-jvm编译过程](/overview/9.Kotlin-jvm编译过程.md)  

  该部分内容主要是包含部分Kotlin代码编译成Java代码这个过程更加深入的理解Kotlin中部分语法知识

* [Kotlin -DSL](/overview/10.Kotlin-DSL.md)  

  该部分内容主要是DSL(Domain Special Language)是**特定领域**语言，与通用语言不通，他只管他的领域，如：SQL、正则表达式；
