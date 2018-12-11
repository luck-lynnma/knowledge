# Object类的扩展

+ `each()`，`collect()`,`find()`,`findAll()`,`any()`和`every()`,不仅collection可以使用，任何对象上都可以使用这些方法
+ 使用上下文`with()`方法：可以更方便的在一个对象上调用多个方法
+ `sleep(millSounds)`:Object方法上的sleep()叫`酣睡`，在给定的毫秒时间内睡眠时，该方法会忽然中断
+ `属性访问属性值`
+ `间接访问属性`  属性编写时不能确定时，可以使用`[]操作符`(对应groovy中的gitAt()),动态访问属性

 ```groovy
 class Car{
    int miles,fuelLevel
}
def propertyTest(){
    def car = new Car(fuelLevel: 80,miles: 25)
    def properties = ["miles","fuelLevel"]
    properties.each {
        name ->
            println("$name=${car[name]}")
    }
    car[properties[1]] = 100
    println "fuelLevel now is ${car.fuelLevel}"
}
 ```

+ `间接调用方法` 编写代码不知道方法名，而在运行时获取。以String形式接收方法，并想调用该方法，使用`反射`来实现，groovy使用`invokeMethod()`,groovy中所有对象都支持该方法。

```groovy
 class Person{
     def walk(){println "Walking...."}
     def walk(int miles){println "Walking $miles miles"}
     def walk(int miles,String where){println "Walking $miles miles $where"}
 }
 def reflectTest(){
     def person = new Person()
     person.invokeMethod("walk",null)
     person.invokeMethod("walk",10)
     person.invokeMethod("walk",[2,'uphill'])
 }
```

+ 数组扩展，所有的数组都可以使用Range对象作为索引

```groovy
 int[] arr = [1,2,3,4,5,6]
 println arr[2..4]
```

+ List、Map扩展  
  1. 使用`[]`获取某个值，`[Range]`获取连续值
  2. 使用`each{}`,内部迭代器，`reverseEach{}`反向迭代元素。 使用 `<<操作符`（映射到leftShift()方法）将所得值写入到结果中

```groovy
 def doubles = []
    def lst = [1,2,3,4,5,6,7]
    lst.each {
        doubles << it * 2
    }
    doubles.each {
        println(it)
    }
```

3. `collect()` 在集合中每个元素上都执行一个操作，并返回一个结果集合

```groovy
 lst.collect {it *2}   //等同于上文的方法
```

4. `find{}、findAll{}`，`find{}`挑出第一个满足条件的元素，`findAll`找出`所有`符合条件的元素
5. `sum()`求和，可以与`collect()`搭配使用，对集合中所有`collect()`操作后的元素求和
6. `join()`迭代每个元素，将每个元素和作为入参给定的字符连接起来

```groovy
 def strings = ["groovy", "Ruby", "Nodejs"]
 println strings.join(' ')  //输出结果：groovy Ruby Nodejs
```

7. Map中`any()`,find()是找到满足符合条件的元素，但要只是确定集合中是否有符合条件的元素，不是为了得到某元素，可以使用`any()`

+ java.lang 的扩展
  + `<< 操作符`：写入内容。
  + `withWriter()`：一旦数据写入流中，就会自动刷新，并关闭该流。将`outputStreamWriter`附到`outputStream`上，并将其传递给闭包，当从闭包返回是，自动刷新并关闭流，闭包出现异常时，也会关闭流
  + `withStream()`使用完自动刷新并关闭流。该方法会调用入参传入的闭包，并将InputStream的实例作为参数发送给该闭包
+ Java.io的扩展
  + `eachFile()`和`eachDir()`为目录和文件的导航进行迭代
  + 为BufferedReader、InputStream和File添加了一个`text`属性，处理打印很方便

```groovy
 println new File().text
```
