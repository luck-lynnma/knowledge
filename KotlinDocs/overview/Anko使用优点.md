# 使用Kotlin和Anko的Android开发

## 什么是Anko

Anko是Kotlin写的Android DSL（Domain-Specific Language）。长久以来，Android视图都是使用XML来完成布局的，这些XML可重用性比较差。同时在运行时，XML要转换为Java表述，这在一定程度上占用了CPU和耗费了资源

Anko是一个Kotlin库，它使Android应用程序的开发变得更快、更容易；使我们代码更干净，易于阅读。

## Anko组成
Anko由以下几个部分组成：

|模块|功能说明|引入对应库|
|:---|:----|:---|
|Anko Commons|使得对intent、dialog、Log、Toast等操作更加简单的轻量级库|compile "org.jetbrains.anko:anko-commons:$anko_version"|
|Anko Layouts|快速和类型安全的动态Android布局库|compile "org.jetbrains.anko:anko-sdk25:$anko_version"<br/>compile "org.jetbrains.anko:anko-appcompat-v7:$anko_version"<br> // 主要为兼容一些控件事件的协程，不过协程coroutines目前还不是kotlin的正式内容<br>compile "org.jetbrains.anko:anko-sdk25-coroutines:$anko_version"<br>compile "org.jetbrains.anko:anko-appcompat-v7-coroutines:$anko_version"|
|Anko SQLite|用于Android sqlite 的查询和分析库|compile "org.jetbrains.anko:anko-sqlite:$anko_version"|
|Anko Coroutines|基于Kotlin协程库|compile "org.jetbrains.anko:anko-coroutines:$anko_version"|


## Anko布局优点

在学习使用Anko进行布局时，我们看到，Anko布局可能会比较复杂，而且又不能实时预览布局，为什么还要推荐使用Anko布局呢？

**因为Anko性能更优**

+ XML布局是运行时解析的。也就是说XML需要从资源文件中获取，然后XmlPullParser解析所有的元素，并一个个的创建它们。还要解析元素的属性，然后设置，这个过程非常繁重，那么到底浪费多长时间在上面呢？

![](http://www.jcodecraeer.com/uploads/20161122/1479791216113147.png)

以上是4个老旧设备上运行测试，虽然老但又是每个安卓开发者都需要处理的设备，结果差距居然高达350%-600%

随着对设计/动画以及性能的需求日益增加，我们大家对速度非常关心，所以为什么使用Anko，答案很明显。

+ 除了布局变快了之外，我们可以加上一些逻辑判断，可以做兼容性检查，根据OS版本设置elevation

```kotlin
coordinatorLayout {
      fitsSystemWindows = true

      appBarLayout {
        toolBar = toolbar {
          if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) elevation = 4f
        }.lparams(width = matchParent, height = actionBarSize())

      }.lparams(width = matchParent)

      container = frameLayout()
        .lparams(width = matchParent, height = matchParent) {
          behavior = AppBarLayout.ScrollingViewBehavior()
        }
    }
```

+ 以上布局我们还可以提取出来，使用Anko提供的AnkoComponent来解决这个问题

### 总结
Anko提供了几种解决传统xml布局构建方式的方法。它绕开了xml布局方式的所有开销，无需处理findViewById或者类型转换，而且在运行时构建布局可以让你添加想要的逻辑，让布局更加动态。所有这些只需要花很小的代价

#
