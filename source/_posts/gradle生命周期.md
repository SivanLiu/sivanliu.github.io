---
layout: _post
title: Gradle 生命周期
date: 2018-02-03 09:03:03
tags: gradle
top: 28
---

直到现在，As 和 gradle 应该算是 Android 开发的标配了，可以极大地提高开发效率。在此过程中，有必要弄清楚 gradle 生命周期，才可以更好地实现自定义编译需求。

gradle 生命周期中的三个重要阶段，也是网上说的比较多的：**初始化 ---> 配置 ---> 执行**

#### 1、构建阶段：
- 初始化： Gradle 支持单工程、多工程，在初始化阶段，Gradle 确定需要构建哪些 project，然后为参与构建的每个工程创建一个 project 实例。

- 配置：此阶段，对 project 对象进行配置，属于构建的所有 projects 的构建脚本都会被执行。

- 执行：Gradle 确定在配置阶段中，创建和配置的要被执行的任务的子集，这个子集是由传递给gradle命令的任务名称参数和当前目录所决定的，然后 Gradle 执行每个选定的任务。

#### 2、settings.gradle 文件：

每个 Gradle 工程中，都会存在一个 settings.gradle，该文件是由 Gradle 通过一种命名约定来决定的，setttings.gradle 会在初始化阶段执行。对于多项目的构建，必须在这个多项目的层次结构中的根目录下有一个 settings.gradle 文件来定义哪些项目会加入这个多项目构建。而对于单项目的构建，有没有 settings.gradle 都可以。

* **单项目构建**：  

**settings.gradle**

```groovy
println('sivan this is a initialization phase')
```
**build.gradle**

```groovy
println('sivan this is a configuration phase')
task configured {
    println 'sivan this is also configuration phase.'
}
task stest << {
    println('sivan this is execution phase')
}
```

**结果:**

```groovy
sivanliu$ gradle stest
sivan this is a initialization phase
sivan this is a configuration phase
sivan this is also configuration phase.
sivan this is execution phase
```

- **多项目构建**：多项目构建是指在一次 Gradle 执行中，构建多个项目，需要在 settings 文件中，声明要加入多项目构建的项目。

- **项目位置**：多项目构建总是由一个具有单个根节点的树表示。树中的每个元素表示一个项目。一个项目有一个路径，表示在多项目构建树中的位置。在大多数情况下，项目路径是符合在文件系统中项目的物理位置的。不过，这种行为是可配置的。项目树在 settings.gradle 文件中创建。默认情况下，它假设设置文件的位置也是根项目的位置。但你可以在设置文件中重新定义根项目的位置。

- **生成树**：在设置文件中你可以使用一套方法来生成项目树。

```groovy
include ':app', ':lib:mylibrary'
```
- 修改项目树元素: 在设置文件中创建的多项目树组成了所谓的项目描述符。你可以在任何时间修改设置文件中的这些描述符。

**settings.gradle:**

```groovy
rootProject.name = 'GradleDemo'
project(':projectA').projectDir = new File(settingsDir, '../app')
project(':projectA').buildFileName = 'projectA.gradle'
```
对于一个构建脚本，属性的访问和方法的调用会委派给一个 project 对象。同样，在设置文件中的属性访问和方法调用也会委托给一个 settings 对象。

#### 3.构建脚本生命周期监听：
在构建通过其生命周期的时候，构建脚本会在特定的阶段接收到对应的通知，通知一般有两种形式︰
1）实现一个特定的监听接口；
2）提供一个用于在收到通知时执行的闭包。

- **gradle 回调闭包监听**：

```groovy
    void beforeProject(Closure var1);
    
    void afterProject(Closure var1);
    
    void buildStarted(Closure var1);
    
    void settingsEvaluated(Closure var1);
    
    void projectsLoaded(Closure var1);
    
    void projectsEvaluated(Closure var1);
    
    void buildFinished(Closure var1);
```

- **project 回调闭包监听**：

```groovy
     /**
     * Adds an action to execute immediately before this project is evaluated.
     *
     * @param action the action to execute.
     */
    void beforeEvaluate(Action<? super Project> action);
    
    /**
     * Adds an action to execute immediately after this project is evaluated.
     *
     * @param action the action to execute.
     */
    void afterEvaluate(Action<? super Project> action);
```

- **测试示例**：

settings.gradle:

```groovy
include ':app', ':mylibrary'
gradle.projectsLoaded {
    println('sssss sivan projectsLoaded '+it)
}
gradle.settingsEvaluated {
    println('sssss sivan settingsEvaluated '+it)
}
gradle.beforeProject {
    println('sssss sivan beforeProject '+it)
}
```

root/build.gradle:

```groovy
allprojects {
    beforeEvaluate {
        println('sssss sivan beforeEvaluate ' + it)
    }

    repositories {
        jcenter()
    }
}
```

app/build.gradle:

```groovy
gradle.projectsEvaluated {
    println('sssss sivan projectsEvaluated '+ it))
}

project.afterEvaluate {
    println('sssss sivan afterEvaluate '+ it)
}
gradle.buildFinished {
    println('sssss sivan buildFinished '+ it))
}

gradle.afterProject {
    println('sssss sivan afterProject '+ it))
}

/**
 * buildStarted 回调在构建开始后，init.gradle 执行前，buildStarted 方法就会被回调，
 * 因此在 build.gradle 加入的监听器 buildStarted 是不会被回调的，只有在 gradle 内部注册的才会回调
 */
gradle.buildStarted {
    println('sssss sivan buildStarted '+it)
}
```

- **结果**：

```groovy
sssss sivan settingsEvaluated settings 'GradleDemo'
sssss sivan projectsLoaded build 'GradleDemo'
sssss sivan beforeProject root project 'GradleDemo'
sssss sivan beforeProject project ':app'
sssss sivan beforeEvaluate project ':app'
sssss sivan afterProject project ':app'
sssss sivan afterEvaluate project ':app'
sssss sivan beforeProject project ':mylibrary'
sssss sivan beforeEvaluate project ':mylibrary'
sssss sivan afterProject project ':mylibrary'
sssss sivan projectsEvaluated build 'GradleDemo'
```

可以看出每个回调只有在特定的阶段执行才会收到回调通知，了解了这些回调的时机之后，就可以根据自己的需求去 hook 指定的任务了。









