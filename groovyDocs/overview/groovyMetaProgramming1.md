# 应用编译时元编程

Groovy同时提供了运行时和编译时元编程。

`运行时元编程`，可以将与类和实例上的方法的拦截、注入、合成决策推迟运行，就元编程而言，大部分情况下，这些便够了；`编译时元编程`是一种高级特性，可用于某些特殊情况，现在主要是框架或工具的编写者使用。

编译时元编程，正是某些强大的特性和基于Groovy工具背后的魔力所在。比如，Groovy 2中的类型检查器就实现了一个抽象语法树(AST)变换。单元测试工具Spock使用这种方式来支持流畅测试用例，该特性支撑起了Groovy的注解。

本节将介绍如何使用`编译时元编程`来分析`代码结构、拦截方法和注入行为`

## 理解代码结构

检查代码以为，需要遍历代码结构，分析类名、方法和参数名等信息。可以编写一个解析器来实现，但是，编译器已经吸并分析过代码，倒不如依靠编译器，尽量减少认为的工作。

编译时元编程容许编译时生成代码，这种转换会影响AST，这也就是我们在Groovy中把它成为AST转换的原因

`编译时元编程性能会好于运行时元编程，因为不需要初始化过程`

## 可用的AST转换

Groovy这种有很多可用的AST转换，它们可满足不同的需求：减少样板文件，实现设计模式，记录日志，实现Swing模式，测试并管理各种依赖等等。更甚者如果没有一个能满足的，还可以`自定义AST转换`

AST转换可分为两大类：

+ `全局AST转换`：它们的应用是透明的，具有全局性的，只要能在类路径上找到它们，就可以使用它们
+ `本地AST转换`：利用标记来注解源代码。与全局AST转换不同，本地AST转换可能支持形式参数

>groovy并不带有任何的全局AST转换，但是可以找到一些可用的本地AST转换

### 代码生成转换

+ @groovy.transfrom.ToString  
  `@ToString` AST转换能够生成人类可读的类的toString形式

```groovy
@ToString
static class Person{
    String firstName
    String lastName
}
def p = new Person(firstName: 'Jack', lastName: 'Nicholson')
println(p.toString()) //输出——包名.Person(Jack,Nicholson)
```

+ @groovy.transform.EqualsAndHashCode  
  `EqualsAndHashCode` AST转换主要是为了生成`equals`和`hashCode`

+ @groovy.transform.TupleConstructor  
  `@TupleConstructor`主要用于通过生成构造函数消除样板文件代码

+ groovy.transform.Canonical  
  `@Canonical` AST转换结合`@ToString`、`@EqualsAndHashCode`、`@TupleConstructor`这三个注解的结果

+ @groovy.transform.InheritConstructors  
  `@InheritConstructor` AST转换意在生成匹配超级构造函数的构造函数。在重写异常类时，这种标记非常有用

+ @groovy.lang.Category  
  前面讲过，用于`分类`注入

+ @groovy.transform.IndexedProperty  
  `@IndexedProperty`标记用于为`列表或数组`类型的属性生成索引化的 getter/setter

+ @groovy.lang.Lazy  
  `@Lazy` AST转换实现了字段惰性初始化

+ @groovy.lang.Newify  
  `@Newify` AST转换用于为构造函数对象提供替代语法

+ @groovy.transform.Sortable  
  `@Sortable`AST转换被用于帮助编写能够实现`Comparable`接口并可按照多种属性快速排序的类

```groovy
@Sortable class Person {
    String first
    String last
    Integer born
}
```

所产生的类具有下列属性：
1. 实现了`Comparable`接口
2. 包含一个`compareTo`方法，以及根据 first、last、born 属性自然排序的一个实现
3. 拥有返回比较器的三个方法：comparatorByFirst、comparatorByLast 和 comparatorByBorn

+ @groovy.transform.builder.Builder

### 类设计注解

这一类注解主要用于简化一些知名模式（委托，单例模式等等）的实现，采用的是一种声明式的风格

+ @groovy.lang.Delegate  
  `@Delegate`AST转换主要用于实现委托设计模式

+ @groovy.transform.Immutable  
  `@Immutable`AST转换简化了不可变类（类成员被认为是不可变的）的创建工作。

```groovy
@Immutable
class Point {
    int x
    int y
}
```

利用`@Immutable`注解生成的不可变类都是`final`类型的类，要想使类不可变，必须确保属性的类型不可变(原始类型或是装箱类型)

+ @groovy.transform.Memoized  
  `@Memoized`AST转化简化了缓存的实现，只需要通过`@Memoized`注释方法，就使方法调用结果能够得到缓存

+ @groovy.lang.Singleton  
  `@Singleton` 注释用于在一个类中实现单例模式

### 日志改进

groovy提供的AST转换可以帮助集成那些广泛使用日志框架

+ @groovy.util.logging.Log  
 首先介绍的日志AST转换是`@Log`注释，依赖的是JDK日志框架
+ @groovy.util.logging.Commons  
 Groovy 支持 `Apache Commons Logging` 框架，用到了 `@Commons` 注释。
+ @groovy.util.logging.Log4j  
 Groovy 还支持 `Apache Log4j 1.x` 框架，使用的是 `@Log4j` 注释
+ @groovy.util.logging.Log4j2  
 Groovy 还支持 `Apache Log4j 2.x` 框架，用的是 `@Log4j2` 注释
+ @groovy.util.logging.Slf4j  
 Groovy 支持 Simple Logging Facade for Java (SLF4J) 框架，使用 `@Slf4j` 注释

 以上所有日志转换的工作方式都差不多：
+ 添加了一个与日志记录器相关的静态final `log`字段
+ 根据底层框架，将所有的对`log.level()`的调用封装为正确的`log.isLevelEnable`防护，这里举一个例子

```groovy
@groovy.util.logging.Log
    static class Greeter {
        void greet() {
            log.info 'Called greeter'
            println 'Hello, world!'
    }
}
//以上代码其实等同于
import java.util.logging.Level
import java.util.logging.Logger

class Greeter {
    private static final Logger log = Logger.getLogger(Greeter.name)
        void greet() {
            if (log.isLoggable(Level.INFO)) {
                log.info 'Called greeter'
            }
        println 'Hello, world!'
    }
}
```

使用`@Commons、@Log4j、@Log4j2、@Slf4j`时注解所做的转换工作基本一样，即使用`注解AST转换`替换了初始化一个static final的log对象，和log.level()防护。

### 声明式转发

groovy提供了一系列注释，以声明式的方式来简化的并发模式

+ @groovy.transform.Synchronized  
 `@Synchronized` AST 转换与 synchronized 关键字运作方式相似，但为了更安全的并发，更关注不同对象。可以用于任何方法或静态方法。
+ @groovy.transform.WithReadLock and @groovy.transform.WithWriteLock  
 `@WithReadLock` AST 转换一般与 `@WithWriteLock` 转换协同使用，利用 JDK 提供的 `ReentrantReadWriteLock` 实现读/写同步功能。

### 更安全的脚本

何为更安全呢？例如执行用户脚本判断脚本不会耗光所有CPU资源(无限循环)或并发脚本不会逐渐耗光线程池中所有可用线程等等。  
JVM经常遇到线程无法停止，或造成一定的麻烦：需要由线程中执行的代码负责检查该标记并正确退出。只有当开发者确切的知道执行的代码要在一个独立的线程中执行，才有意义，但一般来说开发者无从得知，更糟的，用户的脚本甚至不知道执行的代码的线程是哪一个。
解决此问题的注解有：

+ @groovy.transform.ThreadInterrupt  
 `@ThreabInterrupt`注解简化了这些操作，在关键结构中添加线程中断检查
+ 循环结构
+ 方法的首指令
+ 闭包的首指令

+ @groovy.transform.TimedInterrupt
 `@TimedInterrupt` AST 转换所要解决的问题与 `@groovy.transform.ThreadInterrupt` 稍微有所不同：不是检查线程的 interrupt 标记，而是当线程运行时间过长时，会自动抛出异常.  
  该注解**不产生**监控线程。它的工作方式类似于`@ThreadInterrupt`：在合适的位置处放置检查，这意味着如果出现I/O导致线程阻塞，线程就不会被终端
+ @groovy.transform.ConditionalInterrupt

还有很多注解，这里不一一进行介绍，详情请看[运行时及编译时元编程](http://wiki.jikexueyuan.com/project/groovy-introduction/runtime-compile-time-metaprogramming.html)

## 开发AST转换

有两类转换：全局和局部

+ `全局转换`由编译器应用于被编译的代码上的。`实现全局转换的已编译类 位于 一个已添加到编译器路径的 jar文件中`。将针对编译时的每一个源来运行，所以为了提高编译速度起见，千万不要使用耗费时间的方式来创建能够扫描所有AST的转换
+ `局部转换`是一种通过注释那些想要转换的代码元素，实现局部应用的转换，这些注释应该实现`org.codehaus.groovy.transform.ASTTransfromation`,编译器自然会发现它们，然后将转换应用于这些代码元素

### AST转换阶段

Groovy编译器包括多个阶段，结果如下：

+ 初始化阶段(Initialization):打开源文件，配置环境参数  

+ 语法解析阶段(Parsing):使用语法来产生表示源代码的令牌树  

+ 转换阶段(Conversion):从令牌树中创建AST(抽象语法树)

+ 语义分析阶段(Semantic Analysis)：针对一致性及有效性进行检查

+ 规范化阶段(Canonicalization):完整构建AST

+ 指令选择阶段(Instruction Selection):选择指令集合

+ 类生成阶段(Class Generation):从内存中创建字节码

+ 输出阶段(Output):编写二进制输出到文件系统

+ 终止阶段(Finalization):执行最后的垃圾回收及清理工作

`全局转换可以应用于任何一个阶段，局部转换只能用于 语义分析阶段或其后阶段`。如果涉及到读取AST，稍后一些的阶段信息会更丰富，如果涉及到写入AST，则采用较早的阶段，因为此时AST结构更稀疏

### 使用AST变换拦截方法

前面已经了解到运行时拦截的方法，因为是作用于运行时，所以会对执行造成轻微影响。这里介绍一种新的拦截方法：`编译时拦截方法`。

#### 全局转换

这里首先介绍`全局转换`，应用到每个被编译的类上，由于全局转换会对编译器效率产生极大地影响，所以不到万不得已不可使用。实现这一点需要完成以下两步工作：

+ 在 `META-INF/services` 目录中创建`org.codehaus.groovy.transform.ASTTransformation` 描述符。
+ 创建 `ASTTransformation` 实现。  
  + 一般为了能提高编译器的性能，强烈建议使用`@CompileStatic`标注变换类
  + 使用`@GroovyASTTransfromation（phase=CompilePhase.）`标注一个变换类，告诉编译器在XX阶段应用该变换
  + 继承与`ASTTransformation`接口声明一个变换类，这个变换类只有一个`visit(ASTNode[] nodes,SourceUnit sourceUnit)`方法
  + visit()方法中第一个参数`ASTNode nodes`，这个参数包含2个AST节点，第一个是注释节点(`@WithLogging`),第二个是被注释的节点
  + 在变换类的visit()方法中可以使用`ExpressionStatement`创建一个表达节点，并插入到目标方法AST的语句列表中。  
 这一步中还有一个更简单的创建节点方法：`ASTBuilder`，这个类有3个创建`AST`子树的方法：`buildFromSpec()、buildFromString()、buildFromCode()`,其中`buildFromCode()`优先使用

以上是创建一个`全局AST变换`的步骤，可以通过在`ASTTransformation`类中的visit()方法获取想要拦截类名(例如是`CheckingAccount`)拦截到之后，创建一个注入方法的节点(例如：audit()方法)，并将audit()方法节点添加到CheckingAccount的AST，便可达到`编译期拦截方法并注入方法`的功能。同时`本地转换`也能实现拦截并注入方法，并且推荐使用局部转换。

#### 本地转换

创建`本地AST变换`步骤：

+ 创建 `ASTTransformation` 实现
+ 新建一个注解，并使用`@GroovyASTTransformationClass(["包名.变换类名"])`标注新建注释，使新建注释与编写的`ASTTransformation`类链接起来。

下面以一个例子来说明：

```groovy
//声明一个@WithLogging
@Retention(RetentionPolicy.SOURCE)
@Target([ElementType.METHOD])
@GroovyASTTransformationClass(["gep.WithLoggingASTTransformation"])
public @interface WithLogging {
}
//使用@WithLogging
@WithLogging
def greet() {
    println "Hello World"
}
//gep.WithLoggingASTTransformation的实现——变换类
@CompileStatic
@GroovyASTTransformation(phase=CompilePhase.SEMANTIC_ANALYSIS)
class WithLoggingASTTransformation implements ASTTransformation {
    @Override
    void visit(ASTNode[] nodes, SourceUnit sourceUnit) {
        MethodNode method = (MethodNode) nodes[1]
        def startMessage = createPrintlnAst("Starting $method.name")
        def endMessage = createPrintlnAst("Ending $method.name")
        def existingStatements = ((BlockStatement)method.code).statements
        //将节点添加到目标方法的AST
        existingStatements.add(0, startMessage)
        existingStatements.add(endMessage)
    }
    //使用ExpressionStatement来创建println方法AST节点，并将该节点插入目标方法的AST列表内，同时可以使用ASTBuild来使节点创建更加简便
    private static Statement createPrintlnAst(String message) { ⑪
        new ExpressionStatement(
            //MethodCallExpression有3个参数，1个参数：此调用执行当前环境中的当前对象，2个参数：方法名称，3个参数：传入这个方法的参数
            new MethodCallExpression(
                new VariableExpression("this"),
                new ConstantExpression("println"),
                new ArgumentListExpression(
                    new ConstantExpression(message)
                )
            )
        )
    }
}
```