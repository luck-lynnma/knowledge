# 迭代——each()

迭代是编程的基础。本文将将诶少Groovy的万能的each()方法，从而能将复杂的Java语言的迭代抛在脑后。在Groovy中只需要直接使用each()方法，并传递一个闭包，就可以遍历集合

## List集合

```groovy
//list集合
def listTest = ["org/gradle/api/**","org/gradle/internal/**"]  //def listTest是Groovy构建ArrayList的便携语法，可以使用print listTest.class进行验证

List<String> list = new ArrayList<>(listTest)
list.add("org/gradle/api2/**")

task listTestTask << {
   listTest.each{
       println it
   }
}
```

##数组迭代
Groovy允许数组和List交替使用each()方法，为了将ArrayList改为String数组，**必须将as String[]添加到行尾** 如下所示：

```groovy
def array = ["Java","Groovy","JavaScript","Kotlin"] as String[] // array就是一个数组
```

## Map迭代
在Java中无法直接迭代Map,在Groovy中，这完全不是问题，具体实现如下：

```groovy
Map<String,String> map = [key1:'value1',key2:'value2']
map.each{
        println "key=${it.key},value=${it.value}"
    }
```

Groovy中Map稍微有点不同，例如，当我们使用 **apply** 方法使用插件时，apply会自动加上Map的一个参数，当我们这样写"apply plugin : 'java'"时,实际上使用的是name参数(name-value)，只不过在Groovy中使用Map没有< >,当方法被调用的时候，name参数就会换成Map的键值对，只不过在Groovy中看起来不像一个Map

## String迭代
现在已经熟悉了each()方法了，它可以出现在所有相关的位置，加入你想迭代一个String，并且是逐一迭代字符，那么马上可以使用each()方法

```groovy
def string = "Hello Groovy"
string.each{
	print "$it"
}
```

## Range 迭代

Groovy提供了原生的Range类型，可以直接迭代，使用 **..** 分隔的所有内容（例如，1..10就是一个Range），迭代方式如上
Range不局限于简单的Integer，Date也是其中的一种

### Date迭代
实现方式如下：

```groovy
def today = new Date()
def nextWeek = today + 7
(today..nextWeek).each{
    println it
}
```

输出结果：

```
Mon May 28 21:49:08 CST 2018
Tue May 29 21:49:08 CST 2018
Wed May 30 21:49:08 CST 2018
Thu May 31 21:49:08 CST 2018
Fri Jun 01 21:49:08 CST 2018
Sat Jun 02 21:49:08 CST 2018
Sun Jun 03 21:49:08 CST 2018
Mon Jun 04 21:49:08 CST 2018
```

可以看出，each()准确地得出想要的结果，还可以根据it获取具体的"year,month,day,hours,minutes,seconds,times"等等具体时间信息

## Enumeration类型
Java中enum是按照顺序保存的随意的值集合。在Groovy中任然可以使用each()方法进行遍历，实现如下：

```groovy
enum DAY {
    MONDAY,TUESDAY,WEDNESDAY,THURSDAY,FRIDAY,SATURDAY,SUNDAY
}

DAY.each{
    println it
}

(DAY.MONDAY..DAY.SUNDAY).each{
    println "today is $it"
}
```

## SQL迭代

在处理关系型数据表时，经常会说“我需要针对表中的每一行执行操作”比较一下前面的例子。很可能会说“我需要对列表中每一种语言执行一些操作”，根据这个道理，groovy.sql.Sql对象提供了一个eachRow()方法，这里先针对Groovy数据库做一点了解，后面会有关于Groovy数据库的详细的介绍文档

## 文件迭代
当完成所有的嵌套BufferedReader和FileReader后（更别提每个流程久违的所有异常处理）很可能已经忘记最初的目的是什么了，代码冗余的厉害，而Groovy中的等效过程如下：

```groovy
//文件迭代
def f = new File("languages.txt")
//file.write和file.text两种写入方式会相互覆盖,所以下面三句中第一句会被第二句覆盖掉，结果只会有后两条字符串
f.write("这是Groovy中file迭代的测试样例\n")
f.text = "Groovy是一种简洁优雅的代码，允许忽略一些在Java中必须的语法元素\n"
f.append( "Groovy中圆括号是可选的，编译器更喜欢这种代码\n")
task fileTask<<{
    // 找出文件路径
    println f.canonicalPath
    f.eachLine{
        println "$it"
    }
}
```

如果处理的是二进制文件，Groovy还提供了一个 **eachByte()** 方法，另外，Java语言中File并不总是一个文件，有时也会是一个目录，Groovy也提供了一些each()已修改目录，下面是一个例子：

```groovy
//目录迭代
def dir = new File(".")
task dirTask<<{
    //eachFile反悔了文件盒子目录，使用Java语言的isFile和isDirectory方法，可以完成更复杂的事情
    dir.eachFile{
        if (it.isFile()){
            println "FILE: $it"
        } else if (it.isDirectory()){
            println "Directory:$it"
        } else {
            println "Uh,I'm not sure what it is ..."
        }
    }
}
```

如果只对 **目录** 感兴趣，可以使用 **eachDir()** 另外还提供了 **eachDirMatch**  **eachDirRecurse**
同样也可以使用each()进行迭代URL和XML文件

# 结束语

在使用each()方法整个过程中，最妙的部分在于它只需要很少的工作就可以处理大量的Groovy内容。了解each()方法后，Groovy中迭代就易于反掌。一旦了解如何遍历List，那么很快就会掌握如何遍历数组、map、String、Range、enum、SQL、ResultSet、File、目录和URL，甚至是XML文档元素
