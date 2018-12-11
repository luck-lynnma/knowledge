# Gradle-Task

## Task介绍

一个Task代表一个构建工作的原子操作，Gradle中，每一个待编译的工程都叫做Project。每一个Project在构建的时候都包含一系列的Task。

+ 一个Task包含若干的Action。所以，Task有doFirst和doLast两个函数，用于添加需要最先执行的Action和需要最后执行的Action。Action就是一个闭包。

+ Task创建的时候可以通过 `type:SomeType`指定Type，Type其实就是告诉Gradle，这个新建的Task对象会从哪个基类Task派生。比如，Gradle本身提供了一些通用的Task，最常见的有Copy任务。Copy是Gradle中的一个类。当我们：`task myTask(type:Copy)`的时候，创建的就是一个Copy Task。

+ 当我们使用task myTask{....}的时候，花括号就是一个闭包（closure）

+ 当我们使用 task myTask << {.....}的时候，我们创建一个Task对象，同时把closure作为一个action加到这个Task的action队列中，并且告诉她“最后才执行这个closure”

## Gradle是什么

+ gradle是一个基于Apache Ant和Apache maven概念的项目自动化构建工具。
+ 它使用一种基于Groovy的特定领域语言(DSL)来声明项目设置，抛弃了基于XML的各种繁琐配置，面向Java应用为主
+ 当前其支持的语言限于Java、Groovy、Kotlin和Scala，计划未来支持更多的语言。基于groovy脚本构建，其build脚本使用groovy语言编写

关于gradle相关运用，可以移步：[Android Gradle使用总结](https://blog.csdn.net/zhaoyanjun6/article/details/77678577)

## groovy是什么

Groovy是一种动态语言，它和Java类似(算是Java的升级版，但是又具备脚本语言的特点)，都在JVM中运行。当运行Groovy脚本时它会先被编译成Java类字节码，然后通过JVM执行这个Java字节码类

关于groovy相关的知识，移步：[Groovy使用完全解析](https://blog.csdn.net/zhaoyanjun6/article/details/70313790)

## Gradle的Project和Task

gradle中最重要的两个概念。每次构建(build)至少由一个project构成，一个project由一到多个task构成。项目结构中的每个build.gradle文件代表一个project，在这编译脚本文件中可以定义一系列的task；task本质上又是一组被顺序执行的Action对象构成，Action其实是一段代码块，类似于Java中方法

## Gradle 生命周期

1. 初始化阶段 ——Initialization  
 这是创建Project阶段，构建工具根据每个build.gradle文件创建出一个Project实例。初始化阶段会执行项目根目录下setting.gradle中的include信息，决定有哪几个工程加入构建，创建project实例，比如下面有三个工程：include ':app',':lib1','lib2'。这是告诉Gradle这些项目需要编译
2. 配置阶段——Configuration  
 这个阶段通过执行构建脚本来为每个project创建并配置Task。会去执行所有工程的build.gradle脚本，为每一个build.gradle文件实例化为一个Gradle的project对象，然后去分析project之间的依赖关系，下载依赖文件，分析project下的task之间的依赖关系。一个对象由多个任务组成，此阶段也会去创建、配置tasks及相关信息
3. 运行阶段——Execution  
 这是Task真正被执行的阶段，Gradle会依赖关系决定哪些Task需要被执行，以及执行的先后顺序。根据gradle命令传递过来的task名称，执行相关依赖任务。task的action会在这个阶段执行

task是gradle中最小的执行单元，我们所有的构建、编译、打包、debug、test等都是去执行某个task，一个project可以有很多个task，task之间可以互相依赖。

Android studio中点击Gradle按钮，会出现一系列task列表，每一个条目都是一个task，这是根据根目录下`build.gradle`中的

```groovy
dependencies{
	classpatch 'com.android.tools.build:gradle:2.2.2'
}
```

还有一个是app目录下的`build.gradle`

```groovy
 apply plugin: 'com.android.application'
```

由以上两段代码决定的。也就是说，Gradle提供了一个框架，这个框架有一些运行的机制可以让你完成编译，但是至于怎么编译是由插件决定的，这就要使用Google提供的Gradle工具

`根目录下的build.gradle中dependencies{classpatch 'com.android.tools.build:gradle:2.2.2'}`是Android gradle编译插件的版本。

`app目录下的build.gradle中的apply plugin:'com.android.application'`是引入Android的应用构建项目，还有`com.android.library`和`com.android.test`用来构建library和测试

所有Android构建需要执行的task都封装在工具里。那么对于开发一个Android应用来说，最关键的部分就是如何来用AndroidGradle插件了。

## 认知Gradle Wrapper

Android studio中默认使用gradle wrapper不直接使用gradle，命令也gradlew而不是gradle

## Android三个重要gradle文件

+ setting.gradle文件件会在构建的Initialization阶段被执行，用于告诉构建系统哪些模块需要包含到构建过程中。
+ 根目录下的build.gradle文件用来配置针对所有模块的一些属性。它默认包含2个代码块：buildscript{...}和allprojects{...}。前者用于配置构建脚本所用到的代码库和依赖关系，后者用于定义所有模块需要需要用到的一些公共属性
+ 模块极配置文件build.gradle针对每个moudle的配置，它有3个重要的代码块：`plugin`，`android`，`dependencies`

## 定制项目属性

 在根目录的build.gradle配置文件中，可以定义适用于所有模块的属性，通过ext代码块来实现。引用这些属性的语法`rootProject.ext.属性名`

## android studio gradle task

|运行task|作用|
|:---|:---|
|gradlew app:clean|移除所有的编译输出文件，比如apk|
|gradlew app:build|构建app moudle,相当于同时执行了check任务和assemble任务|
|gradlew app:check|执行lint检测编译|
|gradle app:assemble|可以编译出release包和debug包，可以使用gradlew assembleRelease或者gradlew assembleDebug来单独编译一种包|
|gradlew app:assembleRelease|app moudle打release包|
|gradlew app:assembleDebug|app moudle打debug包|
|gradlew app:installDebug |安装 app 的 debug 包到手机上|
|gradlew app:uninstallDebug  |卸载手机上 app 的 debug 包|
|gradlew app:uninstallRelease  |卸载手机上 app 的 release 包|
|gradlew app:uninstallAll  |卸载手机上所有 app 的包|

### lint检测

忽略编译器lint 检查

```groovy
android{
	lintOptions{
		abortOnError false
	}
}
```

### buildTypes定义了编译类型

```groovy
android{
  buildTypes {
        release {
            minifyEnabled true  //打开混淆
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
        debug {
            minifyEnabled false //关闭混淆
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
 }
```

### productFlavors多渠道打包

+ AndroidManifest.xml里设置动态渠道变量

```xml
<meta-data
android:name="UMENG_CHANNEL"
android:value="${UMENG_CHANNEL_VALUE}"/>
```

+ build.gradle 设置productFlavors，这里假定我们需要打包的渠道为酷安市场、360、小米、百度、豌豆荚

```xml
android{
  productFlavors{
  kuan {
		  manifestPlaceholders = [UMENG_CHANNEL_VALUE: "kuan"]
	  }
	  xiaomi {
		  manifestPlaceholders = [UMENG_CHANNEL_VALUE: "xiaomi"]
	  }
	  qh360 {
		  manifestPlaceholders = [UMENG_CHANNEL_VALUE: "qh360"]
	  }
	  baidu {
		  manifestPlaceholders = [UMENG_CHANNEL_VALUE: "baidu"]
	  }
	  wandoujia {
		  manifestPlaceholders = [UMENG_CHANNEL_VALUE: "wandoujia"]
	  }
  }
}
```

或者批量修改

```Java
android {
	productFlavors {
		kuan{}
		xiaomi {}
		qh360 {}
		baidu {}
		wandoujia {}
	}
	productFlavors.all {
		flavor -> flavor.maifestPlaceholders = [UMENG_CHANNEL_VALUE:name]
	}
}
```

这样打包的时候就可以选择渠道

![打包](/overview/imgs/product_flavors.png)

或者用命令打包，比如：

```groovy
gradlew assembleWandoujiaRelease  //豌豆荚release包
gradlew assembleWandoujiaDebug //豌豆荚debug包
```

### 多渠道设置包名

有时候我们需要分渠道设置applicationId、友盟appkey、友盟渠道号

```groovy
productFlavors{
	google{
		applicationId "com.wifi.cool"
		manifestPlaceholders = [
			UMENG_APPKEY_VALUE : "456789456789",
			UMENG_CHANNEL_VALUE : "google",
		]
	}
	baidu{
		applicationId 'com.wifi.hacker'
		manifestPlaceholders = [
			UMENG_APPKEY_VALUE  : "123456789789",
			UMENG_CHANNEL_VALUE : "baidu"
		]
	}
}
```
