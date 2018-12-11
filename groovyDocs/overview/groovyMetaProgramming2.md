# 运行时元程序

## 元编程是什么

Java中使用`反射`来探索运行时程序结构，包括程序的类、类的方法、方法接受的参数。  
groovy使用`元编程`达到动态的添加方法修改方法，代码变得更灵活，开发效率也会提高，`使用元编程实现自动化定制`

`元编程意味着编写能够操作程序的程序，包括程序自身`，Groovy通过`MOP（MetaObject Protocol）元对象协议`提供了这种能力。利用groovy的MOP，创建类、编写单元测试和引入模拟对象都很容易

## 云编程干什么

在groovy中，使用MOP可以动态调用方法，甚至在运行时合成类和方法。借助MOP，在groovy中可以创建内部的领域特定语言。groovy生成器也依赖MOP。因此MOP是我们学习和探索的最重要的概念之一

本节从`groovy对象组成`和`groovy对象如何解析Java对象和groovy对象方法调用`

## groovy对象

groovy对象可分为3类：`POJO（Plain Old Java Object）`、`POGO（Plain Old Groovy Object）`、`拦截器`。
POJO是Java中普通对象。POGO是用Groovy编写的对象，扩展了`java.lang.Object`同时也实现了`groovy.lang.GroovyObject`。groovy拦截器是扩展了`GroovyInterceptable`的groovy对象，具有方法拦截的功能

Groovy中可以实现MOP，是因为GroovyLand中每一个对象隐式的实现了`groovy.lang.GroovyObject`

GroovyObject接口：

```groovy
public interface GroovyObject {
    //在调用对象中不存在的方法时触发
    Object invokeMethod(String var1, Object var2);
    //对象属性的每次读取，都会被getProperty()拦截
    Object getProperty(String var1);
    //对象属性写操作都会被setProperty()拦截
    void setProperty(String var1, Object var2);
    //获取一个对象的metaClass
    MetaClass getMetaClass();
    //设置为自己实现的MetaClass来覆盖默认方法拦截
    void setMetaClass(MetaClass var1);
}
```

`invokeMethod()`、`get'Property()`、`setProperty()` 使Java对象有高度的动态性，使用它们可以 `处理运行时创建的方法和属性`；`getMetaClass()`、`setMetaClass()`

GroovyInterceptable接口：

```groovy
public interface GroovyInterceptable extends GroovyObject {
}
```

Groovy支持对POJO和POGO进行元编程。对于POJO，groovy维护了MetaClass的一个MetaClassRegistry，POGO有一个到其MetaClass的直接引用

当我们调用一个方法时，Groovy会检查目标对象是一个POJO还是POGO，不同的对象，Groovy处理的方法不同

+ POJO

  对于POJO，groocy回去应用类的`MetaClassRegistry`取它的MetaClass，并将方法调用委托给它。因此，在它的MetaClass上定义的任何拦截器或者方法，都优先于POJO原来的方法

+ POGO

  对于POGO，groovy过程会较为复杂一些。
  + 该类是否实现了`GroovyInterceptable`，若继承了则调用GroovyInterceptable的`invokeMethod()`
  + 否则判断`该方法是否存在MetaClass或该类`中，若是便调用拦截器或是原来的方法
  + 否则判断是否有`以该方法命名的属性`，是的话如果该属性是`闭包`，则直接调用`闭包的call()`，否则转下一步
  + 否则判断是否有`methodMissing()`,有的话，调用它的methodMissing
  + 否则调用`invokeMethod()`同时抛出`MissingMethodException`异常

## 查询方法和属性

+ 查询方法

  运行时，查询一个对象的方法使用MOP的`getMetaMethods()`来获取一个元方法(MateClass扩展了MetaObjectProtocol)，使用MOP的`getStaticMetaMethods()`获取一个static方法。使用MOP的`getMetaMethods()`和`getStaticMetaMethods()`获取重载方法列表

+ 查询属性

  使用`getMetaProperty()`和`getStaticMetaProperty()`获取一个元属性。

+ 检查方法或属性书否存在

  使用`respondsTo()`检查方法是否存在；使用`hasProperty()`检查属性是否存在

```groovy
class SomeGroovyClass {
    def field1 = 'ha'
    def field2 = 'ho'
    def getField1() {
        return 'getHa'
    }
    def field
    String property1
    void setProperty1(String property1) {
        this.property1 = property1
    }
}
def someGroovyClass = new SomeGroovyClass()
//method.invoke(owner,args),表示执行owner中带有参数args的method方法
someGroovyClass.metaClass.getMetaMethod("getField1").invoke(someGroovyClass)
someGroovyClass.metaClass.getAttribute(someGroovyClass, 'field1')
someGroovyClass.metaClass.setAttribute(someGroovyClass, 'field', 'ha')
someGroovyClass.metaClass.setAttribute(someGroovyClass, 'property1', 'ho')
someGroovyClass.metaClass.getMetaMethod("setProperty1","test").invoke(someGroovyClass,"test")
```

## 使用MOP拦截方法

`AOP—Aspect-Oriented Programming—面向方向编程`，比如方法拦截或方法建议。方法建议包括：前置建议、后置建议、环绕建议。groovy中使用`MOP`就可以实现这些建议或拦截器，不需要使用任何复杂的工具或框架。

groovy方法拦截方式：

+ 对象拦截，实现GroovyInterceptable接口
+ MetaClass拦截，是动态拦截
+ 使用分类拦截

### 使用GroovyInterceptable拦截方法

按照之前的groovy调用POGO方法的流程看，如果一个类实现了GroovyInterceptable的接口，无论调用的方法是否存在，都会调用`invokeMethod()`，换句话说`GroovyInterceptable的invokeMethod()挟持了该对象上的所有方法调用`

```groovy
class Car implements GroovyInterceptable{
    def check(){
        System.out.println ( "check called...")
    }

    def start(){
        System.out.println ("start called...")
    }

    def drive(){
        System.out.println ("drive called...")
    }

    def invokeMethod(String name,args){
        System.out.print ("Call to $name intercepted...")
        if (name != 'check'){
            System.out.print ( "running filter...")
            Car.metaClass.getMetaMethod('check',null).invoke(this,null)
        }
        def validMethod = Car.metaClass.getMetaMethod(name,args)
        if (validMethod){
            validMethod.invoke(this,args)
        } else {
            Car.metaClass.invokeMethod(this,name,args)
        }
    }
}

def car = new Car()
    car.start()
    car.drive()
    car.check()
    try {
        car.speed()
    } catch (Exception e){
        println e
    }
```

结果如下：

```groovy
Call to start intercepted...running filter...check called...
start called...
Call to drive intercepted...running filter...check called...
drive called...
Call to check intercepted...check called...
Call to speed intercepted...running filter...check called...
groovy.lang.MissingMethodException: No signature of method: com.lynnma.groovy.test1.Car.speed() is applicable for argument types: () values: []
Possible solutions: sleep(long), sleep(long, groovy.lang.Closure), split(groovy.lang.Closure), check(), start(), find()
```

可以看出，对象一旦实现了GroovyInterceptable接口，对象上所有的方法调用都会被其`invokeMethod()`方法拦截。

这种方法适用于拦截作者自己的类中的方法。如果要拦截源代码，或者Java类代码，这种方式便不再适用了

### 使用MetaClass拦截方法

```groovy
Integer.metaClass.invokeMethod{
        String name, onjects ->
            System.out.println("Call to $name intercepted on $delegate...")
            def validMethod = Integer.metaClass.getMetaMethod(name,args)
            if (validMethod){
                System.out.println("running pre-filter...")
                def result = validMethod.invoke(delegate,args)
                System.out.println("running post-filter...")
                result
            } else {
                Integer.metaClass.invokeMissingMethod(delegate,name,args)
            }
    }

    println 5.floatValue()
    println 5.intValue()
    try{
        println 5.empty()
    }catch (Exception e){
        println e
    }
```
输出结果：

```groovy
Call to floatValue intercepted on 5...
running pre-filter...
running post-filter...
5.0
Call to intValue intercepted on 5...
running pre-filter...
running post-filter...
5
Call to empty intercepted on 5...
groovy.lang.MissingMethodException: No signature of method: java.lang.Integer.empty() is applicable for argument types: () values: []
Possible solutions: every(), every(groovy.lang.Closure), upto(java.lang.Number, groovy.lang.Closure), wait(), any(), next()
```

以闭包的形式实现了`invokeMethod()`并将起诉设置到Car的MetaClass上。有两点不同需要注意：

+ `invoke()`中使用的是`delegate`，在闭包中`delegate`指向的是要拦截其方法的目标对象
+ 最后调用`invokeMissingMethod()`

MetaClass拦截调用有一个好处，那就是POJO上也可以调用

## MOP方法注入

`方法注入`指可以动态的想类中添加方法，允许在运行时改变行为。这样就不必在使用一个静态结构和一组预先定义好的方法，对象变得敏捷且灵活。`能够修改类的行为，是元编程和MOP的核心`

Groovy的MOP支持以下3种输入行为中任何一种：

+ 分类（category）
+ ExpandoMetaClass
+ Mixin

### 使用分类注入

`分类`是一种修改MetaClass的对象，并且修改自在代码块和执行线程中有效，退出代码块时，一切恢复原状。
举例说明，如果有一个保存在String或StringBuffer中的社会保险号，现在想注入一个toSSN()，负责以`xxx-xx-xxxx`格式返回字符串。

+ 首先想到的是扩展StringBuffer类`SSNStringBuffer`，但是在StringBuffer类中没有toSSN()方法，而String是一个final类不能被继承
+ `分类注入` 一个分类StringUtil，一个use闭包代码块，注入的方法就在代码块内生效，代码块外便会失效

```groovy
class StringUtil{
    //toSSN()内self没有定义参数类型，默认为Object，可以作用于任何对象
    def static toSSN(self){
        if (self.size() == 9){
            "${self[0..2]}-${self[3..4]}-${self[5..8]}"
        }
    }
}

// 代码块内调用String或StringBuffer实例上的toSSN()，会被路由到分类StringUtil中的静态方法。
// toSSN()内self参数被指派为目标实例，指向方法调用的目标
use(StringUtil){
        println "123456789".toSSN()
        println new StringBuffer("987654321").toSSN()
}
```

Groovy的`分类注入`，要求注入方法是`静态方法，并且至少接受一个参数，第一个参数指向方法的调用者。要注入方法所需参数都放在后面`，参数可以是`对象或是闭包`  
另外，我们也可以使用`@Category注解`来帮我们实现分类里的静态方法，`@Category()必须指定类型`用于生成对应类型的toSSN()静态方法

```groovy
@Category(Object)
class StringUtilAnnotated{
    def toSSN(){
        if (size() == 9){
            "${this[0..2]}-${this[3..4]}-${this[5..8]}"
        }
    }
}

use(StringUtilAnnotated){
    println "123456789".toSSN()
    println new StringBuffer("987654321").toSSN()
}
```

### ues()方法调用背后的魔法

groovy将脚本中调用`ues()方法`路由到StringUtil或StringUtilAnnotated的`public static Object use(Class categoryClass,Closure closure)方法`，该方法定义了一个新的作用域，包括栈上的新的方法和属性，用于目标对象的MetaClass，之后检查给定分类中每一个静态方法，并将方法与及其参数加入到属性/方法列表中，之后调用use后的闭包，该闭包内的所有方法调用都会被拦截，然后发送给`分类提供的实现`。一旦`闭包返回，use()结束掉之前创建的作用域，丢弃分类中注入的方法`

### 闭包作为分类注入方法的参数

```groovy
@Category(String)
class FindUtil{
    def extractOnly(closure){
        def result = ''
        this.each {
            //表示Closure的类型是Object->boolean
            if (closure(it)){ result += it }
        }
        result
    }
}

use(FindUtil){
    println "121254123".extractOnly{it == '4' || it == '5'}
}
```

### use()可以接受多个分类

```groovy
use(StringUtilAnnotated,FindUtil){
        println "123456789".toSSN()
        println new StringBuffer("987654321").toSSN()
        println "121254123".extractOnly{it == '4' || it == '5'}
    }
```

虽然use()可以接受多个分类组成的列表，但是还是建议使用多参，各参数间使用`，`隔开的形式。因为groovy会将该类转变为对该类的Class元对象的引用，例如`String == String.class`   
在多个分类作为参数，冲突时，`优先级最高的是列表中最后一个分类`

### 分类注入优缺点

+ 优点：
  + 效果包含在use()代码块中，一旦离开代码块，注入的方法就消失，像扩充了所接收对象的类型
+ 缺点：
  + 有局限性，作用域只在use()代码块中，即限于执行线程
  + 多次进入退出use()是有代价的，每次进入groovy检查分类的静态方法，并将其加入到新作用域的方法列表中，use()结束还要清理该作用域

如果调用不频繁，而且想要分类这种可控的方法注入所提供的隔离性，可以使用分类，否则使用`ExpandoMetaClass`

### 使用ExpandoMetaClass注入方法

+ `修改和增强类的结构`
+ 向类的MetaClass添加方法可以实现向类中注入方法

使用MetaClass注入方法，是全局可用的，也可以向POGO和POJO注入方法

```groovy
Integer.metaClass.daysFromNow = {
    ->
        Calendar today = Calendar.instance
        //delegate表示闭包的接受者，这里指Integer
        today.add(Calendar.DAY_OF_MONTH,delegate)
        today.time
}
    println(2.daysFromNow())
```

以上表示向Integer的MetaClass类中添加了一个daysFromNow()方法，同样也可以定一个属性，那么在Integer的MetaClass上需要定义一个getXXX()的方法。这里只是为Integer注入了一个daysFromNow()方法，如果我也想在Short或是Long等等数据类型上使用daysFromNow()方法.

这里有两个解决方法：

+ 定义一个daysFromNow(),将该方法赋值给Short.metaClass.daysFromNow = daysFromNow; Long.metaClass.daysFromNow = daysFromNow

  ```groovy
   daysFromNow = { ->
      Calendar today = Calendar.instance
      //delegate表示闭包的接受者，这里指Integer
      today.add(Calendar.DAY_OF_MONTH,delegate)
      today.time
     }
   Short.metaClass.daysFromNow = daysFromNow
   Long.metaClass.daysFromNow = daysFromNow
  ```

+ 直接使用Short和Long等对象的超类Number即可

### 向类中注入static方法

```groovy
Integer.metaClass.'static'.isEven = {val -> val % 2 ==0}
println("is 2 even? ${Integer.isEven(2)}")
println("is 3 even? ${Integer.isEven(3)}")
```

### 向类中注入构造器

```groovy
Integer.metaClass.constructor << {
        Calendar calendar ->
             new Integer(calendar.get(Calendar.DAY_OF_YEAR))
    }
    println(new Integer(Calendar.instance))
```

>注意：我们是添加一个构造器，不是替换，所以使用了`<<操作符`  ，要是想要替换一个原来的构造器，可以使用`=操作符`替换`<<操作符`

为Integer注入一个构造器，入参是Calendar  
在注入构造器中，使用了Integer中现有的一个接受int的构造器(calendar.get结果)，不是创建一个新的Integer实例。这种情况下，自动装箱会负责创建一个Integer实例，务必确认该实现没有递归调用自身，否则会导致StackOverflowError

### 向MetaClass添加一堆方法

```groovy
Integer.metaClass{
        daysFromNow = {
            ->
            Calendar today = Calendar.instance
            today.add(Calendar.DAY_OF_MONTH,delegate)
            today.time
        }

        getDaysFromNow = {
            ->
            Calendar today = Calendar.instance
            today.add(Calendar.DAY_OF_MONTH,delegate)
            today.time
        }

        'static' {
            isEven = {val -> val%2 == 0}
            isEven3 = {val -> val%3 == 1}
        }

        constructor = {
            Calendar calendar ->
                new Integer(calendar.get(Calendar.DAY_OF_YEAR))
        }
        constructor ={
            int val ->
                println "Intercepting constructor call"
                constructor = Integer.class.getConstructor(Integer.TYPE)
                constructor = newInstance(val)
        }
    }
```

### 向具体实例中注入方法

之前都是向类中添加方法，groovy也支持向实例中注入方法。注入的方法作用域仅限于具体实例。有两种实现方式：

+ 每个实例都有一个MetaClass，获取某`实例的MetaClass`，并将方法注入到某实例的MetaClass

+ 初始化一个ExpandoMetaClass实例(emc)，将方法加入其中，然后对其初始化（emc.initialize()用于说明方法/属性添加完成）,将emc赋值给`实例的metaClass`

```groovy
// 实例的MetaClass注入方法
def jeck = new Person()
def paul = new Person()
jeck.metaClass.sing = { ->
    "jeck.metaClass oh baby baby...."
}
println(jeck.sing())
try {
    println(paul.sing())
} catch (Exception e){
    println e
}
// 实例化ExpandoMetaClass实例化注入参数
def emc = new ExpandoMetaClass(Person)
emc.sing = { ->
    "emc oh baby baby...."
}
//初始化，说明方法/属性注入完成
emc.initialize()
def lily = new Person()
lily.metaClass = emc
println(lily.sing())
```

去掉这些注入的方法，很简单，只需要 `实例.metaClass = null`,即只需要将MetaClass属性设置为null

### 使用Mixin注入方法

Java中多接口继承，但只允许继承一个类。groovy也遵循这种语义，但是支持将其他的类中实现拉进来。
`Mixin`是一种运行时能力，可以将多个类中的实现进入进来或混入。可以将一个类混入多个类中。也可以将多个类混入到一个类中。实现方法：

使用`@Mixin(Class)`注解，会将作为参数提供的类(Class)中的方法添加到被注解的类中,可以混入多个类

+ 直接使用`@Mixin(Class)`注解

+ 使用特殊的`mixin()`方法

```groovy
@Mixin(Friend)
 class Person{
    String firstName
    String lastName
    String getName(){"$firstName $lastName"}
}
 class Friend{
    def listen(){
        "$name is Listening as a friend"
    }
}
```

>注意：@Mixin方法在groovy新版本中被`deprecated`

另外在没有源代码供我们`@Mixin()`时，也可以使用`Person.mixin(Friend)`将Friend类内方法Mixin注入到Person中

### 在类中使用多个Mixin

混入多个类而导致冲突时，最后加入到Mixin中的方法会隐藏掉已经注入的方法

### 接口中注入方法

所有实现给接口的类都可以使用这个方法

## MOP方法合成

添加行为可分为两类：注入+合成

+ `方法注入`：编写代码时知道想要添加到一个或多个类中的方法的名字。利用方法注入，可以`动态地向类中添加行为`。也可以向类中注入一组实现某一特定功能的可复用的方法，就行`工具函数`，可以使用`分类`、`ExpandoMetaClass`和`Mixin工具`来注入方法

+ `方法合成`：**在调用时动态的确定方法的行为。** groovy的`invokeMethod()`、`methodMissing()`、`GroovyInterceptable`对于合成非常有用。合成方法`可能知道调用时才会作为独立的方法存在`，当调用了一个不存在的方法时，groovy可以拦截该调用

这节中介绍如何向`类`和`具体的实例`添加动态方法

### 使用methodMissing合成方法

之前，我们可以实现`methodMissing()`拦截不存在的方法调用。同样通过实现`propertyMissing()`拦截不存在的属性的访问。在这些方法中，可以动态的为这些不存在的方法或属性实现相应逻辑，然后可以将这些逻辑保存至MetaClass中，重复调用时便会提高效率

#### methodMissing与GroovyInterceptable

对于实现了GroovyInterceptable的对象，调用该对象上的任何方法，都会调用到invokeMethod()。与invokeMethod()不同的是，只有调用不存在的方法时，才会调用到methodMissing()，如果一个对象实现了GroovyInterceptable，不管被调用的方法是否存在，invokeMethod()都会被调用(如果存在的话)。只有对象将控制转移给其他MetaClass的invokeMethod()时，methodMissing()才会被调用

### 使用ExpandoMetaClass合成方法

对于我们无权编辑的源文件的类或者是POJO，就要使用ExpandoMetaClass来合成方法。不同于为每个领域提供一个拦截器，这里将在MetaClass上实现methodMissing()方法。ExpandoMetaClass的invokeMethod()可以拦截方法调用。所以可以混用`invokeMethod()`和`methodMissing()`来拦截对现有的方法和合成方法的调用

#### invokeMethod与methodMissing的对比

invokeMethod()是GroovyObject的一个方法。methodMissing()则是groovy后来引入的，基于MetaClass的处理部分。如果是处理不存在的方法的调用，因该实现methodMissing(),因为它的开销低，如果只是拦截所有方法调用，而不管方法存在与否，则应该使用invokeMethod

### 具体实例合成方法

+ 创建ExpandoMetaClass实例(emc),为emc创建methodMissing方法，emc.initialize()

+ 具体实例.metaClass = mec

只需要以上两个步骤即可

## MOP 汇总

+ `使用Expando创建动态类`  

可以使用`方法/属性注入`，来`动态的创建一个类`,同时也可以`实现委托`

+ 用于方法拦截

使用`GroovyInterceptable`、`ExpandoMetaClass`。有权修改代码的话实现`GroovyInterceptable`，实现`invokeMethod`拦截。
如果无法修改类，或者那是个Java类，使用`ExpandoMetaClass`，因为一个`invokeMethod`可以负责拦截类上的任何方法。

+ 方法注入

使用`分类`和`ExpandoMetaClass`来实现。`分类`完全可以跟`ExpandoMetaClass`相媲美，如果使用分类，可以控制注入方法的位置，但是`分类`有控制，必须在use()代码块中使用，同事限制在执行线程上。如果向在任意位置注入方法，或者想注入静态方法或构造器，`ExpandoMetaClass`是一个好办法

+ 方法合成

如果有权修改代码，能够再类上为想要合成的方法实现methodMissing()方法，可以通过在第一次调用时注入方法来改进性能。同时需要拦截方法，实现`GroovyInterceptable`接口即可

如果无法修改类，或是那是个Java类，可已将methodMissing()方法添加到类的`ExpandoMetaClass`中，同时需要拦截方法，可以在`ExpandoMetaClass`上实现invokeMethod()

元编程是Groovy得以大放异彩的一个关键特性。在运行时扩展程序，以及利用该语言的动态能力。目前为止，所学的元编程技术，为我们提供了在`运行时修改程序`行为的能力。下一篇学习`编译时修改程序`