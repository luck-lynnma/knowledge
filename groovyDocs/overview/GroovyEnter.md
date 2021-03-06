# Groovy入门

## 为什么要学习Groovy

即使Groovy与Java艳艳有很多相似之处，它仍然是另一个语言。那么为什么应该花时间学习它。简单的回答就是：groovy是一种更有生产力的语言。它具有松散的语法和一些特殊功能，能够加快编码速度，而且Groovy比Java更有助于敏捷开发

## 入门非常容易

无需安装新的运行时工具或专门的IDE。实际上，只需要将Groovy的一个Jar文件放在类路径中即可。而且，Groovy是一种开源语言，由Java开发人员社区管理

## Groovy和Java语言对比

### 用Java编写的Hello World

用java编写的典型的Hello World实例如下：

```java
public class HelloWorld {
  public static void main(String[] args) {  
    System.out.println("Hello World!");
  }
}
```

### 编译和运行Java示例

使用javac编译以上编写的类，如下所示：

```java
 c:>javac HelloWorld.java
```

最后，运行经过编译的类：

```java
c:java HelloWorld
```

### 用Groovy编写Hello World

Groovy语法松散，而且，Groovy使日常的编码活动变得更容易。上面Java代码用Groovy编写Hello World程序就如下面这样简单：

```groovy
println "Hello World"
```

>注意，这段代码周围没有类结构，而且也没有方法解构，我们使用println代替了System.out.prinntln

### 运行Groovy代码

运行groovy代码有两种方法：

+ 对源代码运行groovy命令，因为`.groovy`文件就是一个`脚本`，所以可以使用命令行执行
+ 类似与Java传统的方法，即显式的编译代码来创建字节码(.class文件)，可以使用groovyc编译器

### 运行Groovy示例

假如我将代码保存在MyFirstexample.groovy内，只需要输入以下代码就可以运行这个示例：

```groovy
c:>groovy MyFirstexample.groovy
```

控制台上输出"Hello World" 所需要的工作就这么多

>你可能注意到，不必编译   .groovy文件。这是因为Groovy是脚本语言。脚本语言的一个特点就是能够在运行时进行解释。  

Groovy允许完全省略编译步骤，不过仍然可以进行编译。如果你想编译代码，可以使用Groovy的编译器groovyc。  

用groovyc编译Groovy代码会产生标准的Java字节码，然后可以通过java命令运行生成的字节码。这是Groovy的一项经常被忽略的关键特性：**用Groovy编写的所有代码都能够通过标准Java运行时编译和运行**

至于运行代码，如果希望更加简洁，甚至还能输入：

```groovy
c:>groovy -e "println 'Hello World!'"
```

会生成相同的结果，而且甚至无需定义任何文件

## 利用联合编译混合使用Groovy和Java

在Java中，当依赖其他的Java类时，如果没有找到字节码，javac将编译它认为必要的任何类。在groovy中呢？javac对groovy没有那么友好。但是groovy中支持groovyc联合编译，当编译groovy代码时，它会确定是否有任何需要编译的Java类，并负责编译它们。因此我们可以自由地在项目中混合使用Java代码和Groovy代码，而且不必要单独执行编译步骤，简单的调用groovyc 就好
