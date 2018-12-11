# 使用Groovy开发之I/O

Groovy提供了一些辅助方法来帮助开发者开发I/O程序，开发过程中你可以使用标准的Java代码，但是Groovy提供了更多的方法来处理files、streams、readers。

## 读取文件

+ `eachLine()`方法按行读取文件

```groovy
def baseDir = "/Users/jiangxiaoma/Documents"
new File(baseDir, 'test.txt').eachLine { line ->
    println line
}
```

在`eachLine`闭包内发生任何异常，该方法都能确保源文件被正常关闭

+ `withReader()`也可用来读取文件，发生异常，reader将自动被关闭

```groovy
 def count = 0, MAXSIZE = 3
 new File(baseDir,"test.txt").withReader { reader ->
     while (reader.readLine()) {
         if (++count > MAXSIZE) {
             throw new RuntimeException('Test.txt should only have 3 verses')
         }
     }
 }
```

+ 读取文件到List

```groovy
 def list = new File(baseDir,'test.txt').collect{it}
```

+ 读取文件到array

```groovy
def array = new File(baseDir,'test.txt') as String[]
```

+ 读取文件到byte[]

```groovy
 def file = new File(baseDir,'test.txt')
 byte[] contents = file.bytes
```

+ File文件中获取InputStream  
 在进行I/O操作时不仅限于文件，事实上，很多操作都依赖于输入输出流，下面例子从File中获取InputStream

```groovy
 def is = new File(baseDir,'test.txt').withInputStream{
     stream ->
     //do something
     }
```

## 写入文件

+ `writer`写文件  
 不仅需要读文件，也要写入文件，写文件使用`writer`

```groovy
new File(baseDir,'test.txt').withWriter('utf-8') { writer ->
    writer.writeLine 'Into the ancient pond'
    writer.writeLine 'A frog jumps'
    writer.writeLine 'Water’s sound!'
}
```

+ `<<操作符` 写文件

```groovy
//原始字符串
new File(baseDir,'test.txt') << '''Into the ancient pond
    A frog jumps
    Water’s sound!'''
```

+ `withOutputStream`输出流写入文件，同样有自动关闭输出流和处理异常

```groovy
new File(baseDir,'data.bin').withOutputStream{
    stream->
    //do something
}
```

## 遍历文件树

+ `eachFile` 遍历文件夹下file
+ `eachFileMatch(~/.*\.txt/)`遍历文件夹下txt文件
+ `eachFileResource`，如果需要处理更深目录层次的文件，或者只显示文件或者文件夹，可以使用eachFileResource

```groovy
def dir = new File("/")
//dir.eachFileRecurse()方法会递归显示该目录下所有的文件和目录
dir.eachFileRecurse { file ->
    println file.name
}
dir.eachFileRecurse(FileType.FILES) { file ->
    println file.name
}
```

+ `traverse()`更复杂的遍历方法

```groovy
dir.traverse { file ->
    //如果当前文件是一个目录且名字是bin，则停止遍历
    if (file.directory && file.name=='bin') {
        FileVisitResult.TERMINATE
    //否则打印文件名字并继续
    } else {
        println file.name
        FileVisitResult.CONTINUE
   }
}
```

## 数据和类的序列化&反序列化

在Java中使用`java.io.DataOutputStream` 和 `java.io.DataInputStream` 进行序列化和反序列化是非常常用的，使用groovy将使之变得更容易，下面是序列化数据到文件 和 文件读取数据进行反序列化

```groovy
boolean b = true
def baseDir = "/home/lynnma/Documents"
String message = 'Hello from Groovy'
def file = new File(baseDir,'test.txt')
//序列化数据到file
file.withDataOutputStream {
    out ->
        out.writeBoolean(b)
        out.writeUTF(message)
}

file.withDataInputStream {
    input ->
        assert input.readBoolean() == b
        assert input.readUTF() == message
}
```

## 程序中执行shell命令

+ groovy提供了简单的方法执行shell命令

```groovy
//`execute()`方法返回了一个java.lang.Process实例
def process = "ls -l".execute()
    println "Found text ${process.text}"

    //逐行处理
    def process2 = "ls -l".execute()
    process.in.eachLine { line ->
        println line
    }
```

+ 执行windows下的`dir`命令

```groovy
def process = "dir".execute()
println "${process.text}"
```

会返回`IOException`,这是因为`dir`命令是windows下的shell命令（`cmd.ext`）不能执行，应该这样写：

```groovy
def process = "cmd /c dir".execute()
println "${process.text}"
```
