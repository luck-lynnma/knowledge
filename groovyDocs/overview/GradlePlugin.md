# 需求

默认的Android打包插件会把apk命名成`module-productFlavor-buildType.apk`,例如`app-official-debug.apk`，并且会把包文件发布到固定的位置：`module/build/outputs/apk`有的时候，这个命名风格并不是你所要的，你也想将apk输出到别的目录，可以通过gradle插件来实现自定义，这个插件的需求是：

+ 输入一个名为nameMap的closure，用来修改apk名字
+ 输入一个名为destDir的String，用于输出位置

## 原理简述

### 插件之于Gradle

Gradle提供了很多官方插件，用于Java、Groovy等工程的构建和打包。同时也提供了自定义插件的机制，让每人都可以通过插件来实现特定的构建逻辑，并可以把这些逻辑打包起来，分享给其他人。

插件的源码可以使用Groovy、Scala、Java三种语言，平时只是使用Groovy和Java，Groovy用于实现与Gradle构建生命周期(如task的依赖)有关的逻辑，Java用于核心逻辑，表现为Groovy调用Java的代码。

另外，还可以使用Eclipse或者Maven进行开发构建，用Java实现核心业务代码，将有利于实现快速迁移。

### 插件打包方式

Gradle的插件有三种打包方式，主要是按照复杂程度和可见性来划分：

+ Build script  
 把插件写在build.gradle文件中，一般用于简单的逻辑，只在该build.gradle文件中可见，常用来做原型调试，本文将主要介绍此类。
+ buildSrc项目  
 将插件源代码放在`rootProjectDir/buildSrc/src/main/groovy`中，只对该项目中可见，适用于逻辑较为复杂，但又不需要外部可见的插件，本文将不介绍，将在后期文档进行补充
+ 独立项目  
 一个独立的Groovy和Java项目，可以把这个项目打包成Jar文件包，一个jar文件包还可以包含多个插件入口，将文件包发布到托管平台上，供其他人使用，本文将着重介绍此类

## gradle plugin开发

### Build script插件

首先来直接 在build.gradle中写一个plugin:

```groovy
class ApkDistPlugin implements Plugin<Project>{

    @Override
    void apply(Project target) {
        target.task('apkdist')<<{
            println 'hello,world'
        }

    }
}
apply plugin: ApkDistPlugin
```

命令行`gradle -p app/   apkdist` 运行结果：

```gradle
> Task :app:apkdist
hello,world
```

这个插件创建了一个名为`apkdist`的task，并在task中打印

插件是一个类，继承自`org.gradle.api.Plugin`接口，重载`void apply (Project project)` 方法，这个方法将会传入使用这个插件的project的实例，这是一个很重要的context

#### 接收外部参数

通常情况下，插件使用方法需要传入一些配置参数，如bugtags的SDK的插件需要接受两个参数：

+ 定义参数配置DSL

```gradle
apkdistconf {
    nameMap { name ->
        println 'hello,' + name
        return name
    }
    destDir 'your-distribution-dir'
}
```

+ 声明参数类—声明一个Groovy，有两个默认值为null的成员变量

```groovy
class ApkDistExtension {
    Closure nameMap = null;
    String destDir = null;
}
```

+ 将参数引入ApkDistPlugin，Project里有一种方法：`project.extensions.create('apkdistconf',ApkDistExtension)`

>注意：`create`方法有两个参数，第一个参数就是在build.gradle文件中进行参数配置的dsl的名字，必须一致；第二个参数，就是参数类的名字

+ 获取并使用参数

在create了extension之后，如果传入了参数，则会携带在project实例中

```groovy
def closure = project['apkdistconf'].nameMap;
closure('wow!');

println project['apkdistconf'].destDir
```

+ 运行结果—`gradle -p app/apkdist`

```groovy
:app:apkdist
hello, world!
hello, wow!
your-distribution-directory
```

### 独立项目插件

代码写到这里，已经不适合再放在build.gradle文件里面了，Build script插件也不是常用的。建立一个独立项目，把代码搬到对应的地方。

理论上Intellij IDEA开发插件要比AndroidStudio方便一点点，因为有对应的groovy moudle的模板。但其实如果我们了解IDEA的项目文件结构，就不会受到这个局限，无非就是一个build.gradle文件和src源码文件夹，src文件夹下，有groovy和resources两个文件夹，最终项目的文件夹结构是这样的：

![plugin](/overview/imgs/gradle_plugin4.jpg)

#### 创建Gradle Moudle

`AndroidStudio`中没有新建类似`gradle plugin`这样的选项的，那我们如何在`AndroidStudio`中编写Gradle插件，并打包出来？

+ 首先，创建一个Android project
+ 然后在新建一个Moudle，这个Module用于开发Gradle插件，同样，Module里面没有`gradle plugin`给你选，但是我们只是需要一个`容器`来容纳我们缩写的插件，因此，你可以随便选择一个moudle（如Phone&Table Moudle或是Android Library），因为接下来我们是将里面大部分的内容删除，所以选择哪一个类型的Moudle不重要
+ 修改项目文件夹
  + 移除`src/main`目录下Java文件夹（因为这个项目中用不到Java代码）
  + 添加`groovy文件夹`（主要的代码文件放在这里）
  + 添加`resources`文件夹，存放用于标识gradle插件的`mate-data`
  + 修改`build.gradle`文件，原来的内容清空，添加如下代码：

```groovy
//去掉 Java plugin
apply plugin: 'groovy'
apply plugin: 'maven'

dependencies {
    //gradle sdk
    compile gradleApi()
    //groovy sdk
    compile localGroovy()
    compile fileTree(dir:'libs',include:['*.jar'])
}

repositories {
    mavenCentral()
}
```

+ 在`main`目录下`groovy目录`创建以`.groovy`后缀的类，这样IDE才会识别

 groovy又是基于Java，因此，接下来创建groovy的过程跟创建Java很类似。在groovy新建包名，如：`com.hc.plugin`然后该包下新建groovy文件，通过`new->file->myPlugin.groovy`来新建名为`MyPlugin`的groovy文件
+ 为了让我们的groovy类申明为gradle插件，新建groovy需要实现`org.gradle.api.plugin`接口。

+ 现在，已经定义好自己的gradle插件类，接下来就是告诉gradle，哪一个是我们自定义的插件类，因此，需要在main目录下新建`resources`目录，然后在resources目录里在新建`META-INF`目录，在META-INF目录里新建`gradle-plugins`目录，最后在gradle-plugins目录里面新建properties文件，注意这个文件的命名，你可以随意命名，但是后面使用这个插件的时候，会用到这个名字，比如。你取名为`com.hc.gradle.properties`,而在其他build.gradle 文件中使用自定义的插件时需要写成。

>注意： `resources/META_INF/gradle-plugins`这个文件夹是强制要求的，否则不能识别成插件

```groovy
apply plugin: 'com.hc.gradle'
```

然后在`com.hc.gradle.properties`文件里面指明你自定义的类

```groovy
implementation-class = com.hc.plugin.MyPlugin
```

#### 打包到本地Maven

前面我们已经定义好了插件，接下来就是要打包Maven库里面去了，你可以选择打包到本地，或者是远程服务器中。

在我们自定义Module目录下的build.gradle添加如下代码：

```groovy
//group和version在后面使用自定义插件的时候会用到
group='com.hc.plugin'
version='1.0.0'

uploadArchives{
    repositories{
        mavenDeployer{
            //提交到远程服务器
            /**
             * repository(url:"http://www.xxx.com/repos"){
             *     authentication(userName:"admin",password:"admin")
             * }
             */
            //本地的maven地址设置为D：/repos
            repository(url:uri('D:/repos'))
        }
    }
}
```

其中，groovy和version后面会用到，我们后面讲。虽然我们定义好了打包地址以及打包相关配置，但是还需要我们让这个打包task执行。点击`AndroidStudio`后侧的`gradle工具`，看到有`uploadArchives`这个Task，双击`uploadArchives`就会执行打包上传。执行完，我们去对应目录下看一下，结果如下：

![打包](/overview/imgs/gradle_plugin1.png)

其中`com/hc/plugin`是我们自定义的`group`指定的，`MyPlugin`是模块的名称，`1.0.0`是版本号(`version`指定)

#### 使用自定义插件

我们在app这个模块中使用自定义插件，因此在app模块下的build.gradle文件中，需要指定`maven地址、自定义插件的名称以及依赖包名`，简而言之，就是在app这个module的build.gradle文件中后面附加如下代码：

```groovy
buildscript {
    repositories {
        maven {//本地Maven仓库地址
            url uri('repos')
        }
    }
    dependencies {
        //格式为-->group:module:version
        classpath 'com.hc.plugin:myplugin:1.0.0'
    }
}
//com.hc.gradle为resources/META-INF/gradle-plugins 下的properties文件名称
apply plugin: 'com.hc.gradle'
```

### 开发只针对当前项目的Gradle插件

前面我们讲了如何自定义gradle插件并且打包出去，可能步骤比较多。有时候，你可能并不需要打包出去，只是在这一个项目中使用而已，那么你无需打包这个过程。

只是针对当前项目开发的Gradle插件相对较简单。步骤之前所提到的很类似，只是有几点需要注意：

+ 新建的Module名称必须为BuildSrc
+ 无需resources目录

目录结构如下所示：

![目录结构](/overview/imgs/gradle_plugin2.jpg)

+ 1.其中，build.gradle内容为：

```groovy
apply plugin: 'groovy'

dependencies {
    compile gradleApi()//gradle sdk
    compile localGroovy()//groovy sdk
}

repositories {
    jcenter()
}
```

+ 2.SecondPlugin.groovy内容为：

```java
package  com.hc.second

import org.gradle.api.Plugin
import org.gradle.api.Project

public class SecondPlugin implements Plugin<Project> {

    void apply(Project project) {
        System.out.println("========================");
        System.out.println("这是第二个插件!");
        System.out.println("========================");
    }
}
```

+ 3.app 的module中使用方式，可以在app的build,gradle下加入

```groovy
apply plugin: com.hc.second.SecondPlugin //不要使用引号，这是引入plugin的目录
```

clean一下，再make project，messages窗口信息如下：

![messages](/overview/imgs/gradle_plugin3.jpg)

由于之前我们自定义的插件我没有在app的build.gradle中删除，所以hello gradle plugin这条信息还会打印.
