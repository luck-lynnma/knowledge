# 协程是什么？
协程的概念其实早就被提出来。下面为大家讲解下协程究竟是怎么来的：

+ 一开始大家想要同一时间执行多个任务，于是就有了 **并发**。从程宇员的角度可以看成是多个队里的逻辑流，内部可以是多CPU并行，也可以是单CPU时间分片

+ 但是一旦有了并发就会出现 **上下文切换** 的问题，运行了一般跑去处理另一件事，我这做了一般的事情怎么保存。**进程** 就是这样抽象出来的一个概念，搭配虚拟内存、进程表之类，用来管理独立的程序运行、切换

+ 后来硬件提升了，一个电脑上有好几个CPU，就可以一个CPU跑一个进程，这就是所谓的 **并行**

+ 但是一并行，进程数一旦提高，大部分系统资源就得用于保存进程切换的状态，后来衍生出 **线程** 的概念，大致意思就是这个地方堵塞了，但还可以其他地方的逻辑流可以计算，不用特别麻烦的切换页表、刷新TLB，只是把寄存器刷新一遍就好。

+ 如果你嫌操作系统调度线程有不确定性，不知道什么时候开始什么时候且走，我们自己在晋城里面手写代码去管理逻辑调度这就是 **用户态线程**

+ 而用户态线程是不可剥夺的，如果一个用户态线程发生了堵塞，就会造成整个进程堵塞，所以进程需要自己拥有调度线程的能力。而如果用户态线程将控制权交给了进程，让进程自己去调度自己，这就是 **协程**

后来我们的内存越来越大，操作系统的调度也越来越智能，就慢慢的没人去研究实现用户态线程、协程这些东西了。

# 为什么又要用协程了

这就得从多线程的效率讲起了。

前面讲由于操作系统的多线程调度越来越智能，硬件设备也越来越好，大幅度的提升了线程的效率，因此正常的情况下线程的效率是高于协程的，而且远高于协程。

那么线程在什么情况下效率是最高的？就是已知run的情况下。但是线程几乎很难一直run的，比如：线程上下文的切换、复杂计算机阻塞、IO阻塞

于是又有人想起写成了，这个可以交给代码调度的东西

# 协程的本质作用

携程实际上就是极大程度的复用线程，通过让线程满载运行，达到最大程度的利用CPU，进而提升应用性能。具体一个例子来说明：在Android上发起一个网络请求

>1.主线程创建一个网络请求任务<br>
>2.通过子线程去请求服务端响应<br>
> 2.1等待网络传递请求，其中包括TCP/IP的一系列过程<br>
> 2.2 等待服务器处理，比如你请求一个列表数据，服务器逻辑执行一次去缓存、数据库、默认数据找到应该返回给你的数据，再将数据回传<br>
> 2.3 有时一系列的数据回传<br>
>3.在子线程中获取服务器返回的数据。将数据转换成想要的格式<br>
>4.在主线程中执行某个回调方法

在第二个步骤，我们会用一个线程池去存放一批创建好的线程复用，防止多次创建线程。但是使用了线程池就会涉及到池中预存多少线程才最合适？少了，会有任务等待空线程，多了会造成浪费内存。这就是线程利用率不高的问题。上面第2步到2.3步线程是处于堵塞不做事的状态，这也造成线程利用率不高的问题所在。

那么换成协程是这么个流程：
>1.主线程创建一个协程，在协程中创建网络请求任务<br>
>2.为协程分配一个执行线程（肯定是子线程），在线程中去请求服务端响应<br>
>2.1.接下来挂起子线程的这个协程，等待网络传递请求，其中包括TCP/IP的一系列过程<br>
>2.2.协程依旧挂起，等待服务器处理，比如你请求一个列表数据，服务器逻辑执行一次去缓存、数据库、默认数据找到应该返回给你的数据，再将数据回传<br>
>2.3协程依旧挂起，又是一系列的数据回传<br>
>3.获取到服务器返回的数据，在子线程中回复挂起的协程，将数据转换成想要的格式<br>
>4.在主线程中执行某个回调方法<br>

上面的这个例子，步骤没发生变化，但是引入了 **协程挂起** 的概念，当线程中的协程发生了挂起，线程依旧是可以继续做其他的事情，而携程过期是一个很轻的操作（其内在只是一次状态机的变更，就是一个switch语句的分支执行）

# 协程使用
上面所说我们明白一点，协程是通过编码实现一个任务。它和操作系统或JVM没有任何关系，它的存在更类似于虚拟线程。

在Kotlin上，使用协程只需要知道两个方法和它们的返回类型，就可以很收敛的用上协程，分别是：

```
fun launch() :Job
fun async() :Deferred
```

## launch方法
launch()以非阻塞当前线程的方式。启动一个新的协程后台任务，并返回一个Job类型的对象作为当前协程的引用

```kotlin
public fun launch(
    context: CoroutineContext,
    start: CoroutineStart = CoroutineStart.DEFAULT,
    block: suspend CoroutineScope.() -> Unit
): Job {
}
```

launch()有3个入参：context、start、block，这些参数分别说明如下：

|参数|说明|
|:----|:-----|
|context:CoroutineContext|协程上下文，不仅可以用于在协程跳转的时刻传递数据，同时最主要的功能：用于协程运行和回复的上下文环境。通常Android再用的时候都是传入一个UI就表示在UI线程启动协程，或者传入CommonPool表示在异步启动协程，还有一个是Unconfined表示不指定，在哪个线程调用就在哪个线程恢复|
|start|协程启动选项，start是一个枚举对象，默认值DEFAULT根据上下文，立即安排coroutine执行<br>LAZY只有在需要的时候，才会开始<br>ATOMIC原子性(不可撤销)根据上下文来安排执行<br>UNDISPATCHED立即执行协程，直到当前线程中的第一个暂停点|
|block|协程真正要执行的代码块，必须是 **suspend** 修饰的挂起函数|

## Job对象

launch()方法返回一个Job对象，Job对象常用的方法有三个，叫 **start、join和cancel** 分别对应了协程的启动、切换至当前协程、取消

### start()
下面是start()方法的使用实例：

```kotlin
fun test() {
	val job = launch(CommonPool,CoroutineStart.LAZY){
		println("hello")
	}
	job.start()
}
```

### join()
join()方法就比较特殊，它是一个suspend方法。**suspend修饰的方法（或闭包）只能调用被suspend修饰过方法(或闭包)。** 方法声明如下：

```
public suspend fun join()
```

因此，join()只能在协程体内部使用，他的功能：切换至当前协程

## async()方法
async()方法也是创建一个协程并启动，甚至连方法的声明都跟launch()方法一模一样。不同的是，async()方法返回值，返回的是一个Deferred对象。这个接口是Job接口的子类。因此上文介绍的所有方法，都可以用于Deferred的对象。Deferred最大的用处在于他特有的一个方法 **await()**

```
public suspend fun await():T
```

await()可以返回当前协程的执行结果，也就是你可以这样写代码：

```
fun test() {
    val deferred1 = async(CommonPool) {
        "hello1"
    }
    val deferred2 = async(UI) {
        println("hello2")
        println(deferred1.await())
    }
}
```

## suspend
一个协程方法（或闭包）必须被suspend修饰，同时suspend修饰的方法（或闭包）只能被suspend修饰过的方法(闭包)调用。

suspend修饰后代码发生了怎样的变化？

我们知道，Kotlin的闭包(lambda)在编译后转换成了内部类的对象，而一个被suspend修饰过得闭包，就是一个特殊的内部类。

kotlin中使用launch启动一个协程，它的协程体就是一个被suspend修饰的闭包，如下：

```
launch(CommonPool) {
        println("Kotlin!")
       delay(300L,TimeUnit.MILLISECONDS)//挂起函数是一liucheng个suspend关键字修饰的函数，称之为挂起函数
    }
```

转换成Java代码后，如下，第三个参数，就是一个function(launch方法传入的闭包)

```
BuildersKt.launch$default((CoroutineContext)CommonPool.INSTANCE, (CoroutineStart)null, (Function2)(new Function2((Continuation)null) {
......
}
```

而如果是一个普通方法被suspend修饰后，则会多出一个参数，例如一个普通test()无参无内容用suspend修饰后被编译成这样：

```
@Nullable
  public static final Object test(@NotNull Continuation var0) {
	 return Unit.INSTANCE;
  }
```

可以看到不论怎样，都会具备一个 **Continuation** 的对象。而这个 **Continuation就是真正的Kotlin的协程。**

## 协程挂起与恢复

### 协程的启动流程
launch()源码是这样的：

```
public fun launch(
    context: CoroutineContext = DefaultDispatcher,
    start: CoroutineStart = CoroutineStart.DEFAULT,
    block: suspend CoroutineScope.() -> Unit
): Job {
    //省略一些列的初始化后
    start(block, coroutine, coroutine)
    return coroutine
}
```

+ start是一个枚举对象，默认值是DEFAULT，这里实际上是调用了枚举的 **invoke()** 方法，invoke()如下：

```
public operator fun <R, T> invoke终将闭包创建成一(block: suspend R.() -> T, receiver: R, completion: Continuation<T>) =
	   when (this) {
		   CoroutineStart.DEFAULT -> block.startCoroutine(receiver, completion)
		   CoroutineStart.UNDISPATCHED -> block.startCoroutineUndispatched(receiver, completion)
		   CoroutineStart.LAZY -> Unit // will start lazily
	   }
```

+ 然后调用startCoroutine()，方法如下：

```
public fun <R, T> (suspend R.() -> T).startCoroutine(
        receiver: R,
        completion: Continuation<T>
) {
    createCoroutineUnchecked(receiver, completion).resume(Unit)
}
```

+ 最终会调用 **createCoroutineUnchecked** ，这是一个扩展方法，声明如下：

```
public fun <R, T> (suspend R.() -> T).createCoroutineUnchecked(
        receiver: R, completion: Continuation<T>
): Continuation<Unit> =
    if (this !is CoroutineImpl)
        buildContinuationByInvokeCall(completion) {
            (this as Function2<R, Continuation<T>, Any?>).invoke(receiver, completion)
        }
    else
        (this.create(receiver, completion) as CoroutineImpl).facade
```

看到这段源码，通过判断 **this** 是不是 **CoroutineImpl** 做不同操作。而 **this** 就是 **suspend R.()->T** 是suspend修饰的闭包，也就是上面launch()的参数传入的闭包。

上文提到suspend修饰的闭包会在编译后变成一个 **Continuation**，而 **CoroutineImpl** 是 **Continuation** 接口的实现。

因此这里调用了 **闭包的create()**，最终将闭包创建成一个 **Continuation** 对象并返回。这也验证了 **Continuation就是真正的kotlin协程**

+ 创建好协程对象后，又会回到`createCoroutineUnchecked(receiver, completion).resume(Unit)` 又会调用协程 **Continuation的resume()方法** ， 而协程的 resume()方法又会调用编译后 suspend闭包转换成的那个类(**Continuation**)里面的 **doResume()**

```
override fun resume(value: Any?) {
        processBareContinuationResume(completion!!) {
            doResume(value, null)
        }
    }
```

### 协程的挂起

明白了启动流程以后，再来看挂起就清晰多了。下面我们看下代码：

```
public final Object doResume(@Nullable Object var1, @Nullable Throwable throwable) {
        Object var5 = IntrinsicsKt.getCOROUTINE_SUSPENDED();//coroutine_suspended表示协程还未执行完，还在执行
        switch(super.label) {
            case 0:
               if (throwable != null) {
                  throw throwable;
               }

               CoroutineScope var3 = this.p$;
               String var4 = "Kotlin!";
               System.out.println(var4);
               TimeUnit var10001 = TimeUnit.MILLISECONDS;
               super.label = 1;
               Object var10000 = DelayKt.delay(300L, var10001, this);
               InlineMarker.mark(2);
               if (var10000 == var5) { //表示协程正在执行
                  return var5;
               }
               break;
            case 1:
               if (throwable != null) {
                  throw throwable;
               }
               break;
            default:
               throw new IllegalStateException("call to 'resume' before 'invoke' with coroutine");
            }

            return Unit.INSTANCE;
         }
```

在switch()内的label变量，他就是标识符，典型的状态机设计。当前在执行的协程有多少个可能的状态就会有多少位。

当前例子中只有两种状态：初始状态，因为async()的调用被挂起

+ 首先看到label=0 时的代码，首先看到是`Object var5 = IntrinsicsKt.getCOROUTINE_SUSPENDED()`，表示协程还未执行完。启动一个协程，如果这个协程是可以立刻执行完的，那就返回结果；否则直接结束当前方法，等待下次状态改变被触发，而这个结束当前方法，处于等待时刻，就是被挂起的时候

### 内部协程的切换

在协程方法async()返回的是 **Deferred** 接口类型的对象，这个接口也继承了 **Job** 接口，是它的子类。

使用下面的例子进行说明：

```
fun test(){
	launch(CommonPool) {
        val job = async(Unconfined) {
            "string"
        }
        println("========${job.await()}")
    }
}
```

本例子中，async()返回的实际是 **DeferredCoroutine** 这个类的对象，它实现了 **Deferred** 接口，更重要的是，它实现了 **await()** 接口方法，还是看代码：

```
private open class DeferredCoroutine<T>(
    parentContext: CoroutineContext,
    active: Boolean
) : AbstractCoroutine<T>(parentContext, active), Deferred<T> {
	suspend override fun await(): T {
        // fast-path -- check state (avoid extra object creation)
        while(true) { // lock-free loop on state
            val state = this.state
            if (state !is Incomplete) {
                // already complete -- just return result
                if (state is CompletedExceptionally) throw state.exception
                return state as T

            }
            if (startInternal(state) >= 0) break // break unless needs to retry
        }
        return awaitSuspend() // slow-path
    }
}
```

**await()** 其实是`awaitInternal()`代理，**它通过一个`lock-free`循环，保证一定等到异常或者是一个`startInternal()`的方法执行完** 才会返回。`startInternal()`方法的作用是在启动`start=LAZY`时，保证协程初始化完成，所以在本例中没有意义。在本例中有意义的是紧跟着这个方法后面调用的 `awaitSuspend()`：

```
private suspend fun awaitSuspend(): T = suspendCancellableCoroutine { cont ->
        cont.disposeOnCompletion(invokeOnCompletion {
            val state = this.state
            check(state !is Incomplete)
            if (state is CompletedExceptionally)
                cont.resumeWithException(state.exception)
            else
                cont.resume(state as T)
        })
    }
```

这个`cont`就是调用`await()`时传入的外部协程的对象。

`disposeOnCompletion()`方法调用`invokeOnCompletion()`返回`DisposableHandle`对象的`dispose()`方法，去等待`job`中的内容执行完。但如果`job`中的代码在`invokeOnCompletion()`方法返回之前就已经执行完，就会返回一个`NonDisposableHandle`对象表示不需要在等待了。

然后执行闭包中的代码，去根据`job`内的代码是否发生了异常去返回对应的结果，这个结果就是`state`。

最终，又由外部协程`cont`调用了父类的`resume()`方法或这`resumeWithException()`方法（出异常时）


### 协程的恢复
最终，与协程的启动流程中提及的一样，`Continuation`中的`resume()`方法会调用到`suspend`闭包转换的类的`doResume()`方法

```
override fun resume(value: Any?) {
    processBareContinuationResume(completion!!) {
        doResume(value, null)
    }
}
```

而这里的参数`value`，就是协程在恢复时候传入的，内部协程执行后结果。

这时候，我们再看状态机中的`label 1`的代码
