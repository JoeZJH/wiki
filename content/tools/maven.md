---
title: "maven"
layout: page
date: 2016-07-07
---
[TOC]

## 关于
Maven 是一个项目管理和构建自动化工具。但是对于我们程序员来说，我们最关心的是它的项目构建功能。
所以这里我们介绍的就是怎样用 maven 来满足我们项目的日常需要。


## 流程
### 创建maven项目
```bash
mvn archetype:generate -DgroupId=com.mycompany.helloworld -DartifactId=helloworld -Dpackage=com.mycompany.helloworld -Dversion=1.0-SNAPSHOT
```
archetype:generate 目标会列出一系列的 archetype 让你选择。 Archetype 可以理解成项目的模型。 Maven 为我们提供了很多种的项目模型，包括从简单的 Swing 到复杂的 Web 应用。我们选择默认的 maven-archetype-quickstart ，是编号 #807，不同版本这个值会不同，不过都会是默认值，
所以按回车就好了。

创建的配置属性对应的意义参考pom文件说明，这些属性是我们在命令行中用 -D 选项指定的。该选项使用 -Dname=value 的格式。

### 构建项目
```bash
cd helloworld
mvn package
```

当你第一次运行 maven 的时候，它会从网上的 maven 库 (repository) 下载需要的程序，存放在你电脑的本地库 (local repository) 中，所以这个时候你需要有 Internet 连接。Maven 默认的本地库是 ~/.m2/repository/ ，在 Windows 下是 %USER_HOME%\.m2\repository\ 。

maven 在 helloworld 下面建立了一个新的目录 target/ ，构建打包后的 jar 文件 helloworld-1.0-SNAPSHOT.jar 就存放在这个目录下。编译后的 class 文件放在 target/classes/ 目录下面，测试 class 文件放在 target/test-classes/ 目录下面。

### 运行项目
为了验证我们的程序能运行，执行下面的命令：
```bash
java -cp target/helloworld-1.0-SNAPSHOT.jar com.mycompany.helloworld.App
```


## 核心概念
### Pom, project object Model.
配置文件，maven根据该文件构建项目，可以继承。各节点的含义
- project：顶级元素
- modelVersion：object model版本，强制
- groupId：项目创建团体或组织的唯一标识，通常是域名倒写，例如 org.apache.maven.plugins
- artifactId ：是项目artifact唯一的基地址名
- packaging ：artifact打包的方式，如jar、war、ear等等。默认为jar。这个不仅表示项目最终产生何种后缀的文件，
  也表示build过程使用什么样的lifecycle。
- version ：artifact的版本，通常能看见为类似0.0.1-SNAPSHOT，其中SNAPSHOT表示项目开发中，为开发版本
- name： 表示项目的展现名，在maven生成的文档中使用
- url：表示项目的地址，在maven生成的文档中使用
- description： 表示项目的描述，在maven生成的文档中使用
- dependencies： 表示依赖，在子节点dependencies中添加具体依赖的groupId artifactId和version
- build： 表示build配置
- parent： 表示父pom

在 POM 中，groupId, artifactId, packaging, version 叫作 maven 坐标，它能唯一的确定一个项目。有了 maven 坐标，我们就可以用它来指定我们的项目所依赖的其他项目，插件，或者父项目。一般 maven 坐标写成如下的格式：`groupId:artifactId:packaging:version`。
像我们的例子就会写成：`com.mycompany.helloworld: helloworld: jar: 1.0-SNAPSHOT`.

我们的 helloworld 示例很简单，但是大项目一般会分成几个子项目。在这种情况下，每个子项目就会有自己的 POM 文件，然后它们会有一个共同的父项目。这样只要构建父项目就能够构建所有的子项目了。子项目的 POM 会继承父项目的 POM。另外，所有的 POM都继承了一个 Super-POM。Super-POM 设置了一些默认值，它遵循了惯例优于配置的原则。所以尽管我们的这个 POM 很简单，但是这只是你看得见的一部分。运行时候的 POM 要复杂的多。 如果你想看到运行时候的 POM 的全部内容的话，可以运行下面的命令：`mvn help:effective-pom`.

### Maven 插件
我们用了 `mvn archetype:generate` 命令来生成一个项目。那么这里的 `archetype:generate` 是什么意思呢？archetype 是一个插件的名字，generate是目标(goal)的名字。这个命令的意思是告诉 maven 执行 archetype 插件的 generate 目标。插件目标通常会写成 `pluginId:goalId`

一个目标是一个工作单元，而插件则是一个或者多个目标的集合。比如说Jar插件，Compiler插件，Surefire插件等。从看名字就能知道，Jar 插件包含建立Jar文件的目标， Compiler 插件包含编译源代码和单元测试代码的目标。Surefire 插件的话，则是运行单元测试。

看到这里，估计你能明白了，mvn 本身不会做太多的事情，它不知道怎么样编译或者怎么样打包。它把构建的任务交给插件去做。插件定义了常用的构建逻辑，能够被重复利用。这样做的好处是，一旦插件有了更新，那么所有的 maven 用户都能得到更新。


### Maven 生命周期
我们用的第二个命令是：mvn package。这里的 package 是一个maven的生命周期阶段 (lifecycle phase )。生命周期指项目的构建过程，它包含了一系列的有序的阶段 (phase)，而一个阶段就是构建过程中的一个步骤。

那么生命周期阶段和上面说的插件目标之间是什么关系呢？插件目标可以绑定到生命周期阶段上。一个生命周期阶段可以绑定多个插件目标。当 maven 在构建过程中逐步的通过每个阶段时，会执行该阶段所有的插件目标。

maven 能支持不同的生命周期，但是最常用的是默认的Maven生命周期 (default Maven lifecycle )。如果你没有对它进行任何的插件配置或者定制的话，那么上面的命令 mvn package 会依次执行默认生命周期中直到包括 package 阶段前的所有阶段的插件目标：
- process-resources 阶段：resources:resources
- compile 阶段：compiler:compile
- process-classes 阶段：(默认无目标)
- process-test-resources 阶段：resources:testResources
- test-compile 阶段：compiler:testCompile
- test 阶段：surefire:test
- prepare-package 阶段：(默认无目标)
- package 阶段：jar:jar

### Maven 依赖管理
之前我们说过，maven 坐标能够确定一个项目。换句话说，我们可以用它来解决依赖关系。在 POM 中，依赖关系是在 dependencies 部分中定义的。在上面的 POM 例子中，我们用 dependencies 定义了对于 junit 的依赖：
```xml
<dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
</dependencies>
```

那这个例子很简单，但是实际开发中我们会有复杂得多的依赖关系，因为被依赖的 jar 文件会有自己的依赖关系。那么我们是不是需要把那些间接依赖的 jar 文件也都定义在POM中呢？答案是不需要，因为 maven 提供了传递依赖的特性。

所谓传递依赖是指 maven 会检查被依赖的 jar 文件，把它的依赖关系纳入最终解决的依赖关系链中。针对上面的 junit 依赖关系，如果你看一下 maven 的本地库（我们马上会解释 maven 库）~/.m2/repository/junit/junit/3.8.1/ ，你会发现 maven 不但下载了 junit-3.8.1.jar，还下载了它的 POM 文件。这样 maven 就能检查 junit 的依赖关系，把它所需要的依赖也包括进来。

在 POM 的 dependencies 部分中，scope 决定了依赖关系的适用范围。我们的例子中 junit 的 scope 是 test，那么它只会在执行 compiler:testCompile and surefire:test 目标的时候才会被加到 classpath 中，在执行 compiler:compile 目标时是拿不到 junit 的。

我们还可以指定 scope 为 provided，意思是 JDK 或者容器会提供所需的jar文件。比如说在做web应用开发的时候，我们在编译的时候需要 servlet API jar 文件，但是在打包的时候不需要把这个 jar 文件打在 WAR 中，因为servlet容器或者应用服务器会提供的。

scope 的默认值是 compile，即任何时候都会被包含在 classpath 中，在打包的时候也会被包括进去。

### Maven 库
当第一次运行 maven 命令的时候，你需要 Internet 连接，因为它要从网上下载一些文件。那么它从哪里下载呢？它是从 maven 默认的远程库(http://repo1.maven.org/maven2) 下载的。这个远程库有 maven 的核心插件和可供下载的 jar 文件。

但是不是所有的 jar 文件都是可以从默认的远程库下载的，比如说我们自己开发的项目。这个时候，有两个选择：要么在公司内部设置定制库，要么手动下载和安装所需的jar文件到本地库。

本地库是指 maven 下载了插件或者 jar 文件后存放在本地机器上的拷贝。在 Linux 上，它的位置在 ~/.m2/repository，在 Windows XP 上，在 C:\Documents and Settings\username\.m2\repository ，在 Windows 7 上，在 C:\Users\username\.m2\repository。当 maven 查找需要的 jar 文件时，它会先在本地库中寻找，只有在找不到的情况下，才会去远程库中找。

运行下面的命令能把我们的 helloworld 项目安装到本地库：
```bash
     $mvn install
```
一旦一个项目被安装到了本地库后，你别的项目就可以通过 maven 坐标和这个项目建立依赖关系。比如如果我现在有一个新项目需要用到 helloworld，那么在运行了上面的 mvn install 命令后，我就可以如下所示来建立依赖关系：

```xml
    <dependency>
      <groupId>com.mycompany.helloworld</groupId>
      <artifactId>helloworld</artifactId>
      <version>1.0-SNAPSHOT</version>
    </dependency>
```
好了，maven 的核心概念就简单的介绍到这里。至于在 Eclipse 中如何使用 maven，这个网上很多了，google 一下就行。

## maven常用参数和命令
- mavn常用参数
```
mvn -e 显示详细错误
mvn -U 强制更新snapshot类型的插件或依赖库（否则maven一天只会更新一次snapshot依赖）
mvn -o 运行offline模式，不联网更新依赖
mvn -N仅在当前项目模块执行命令，关闭reactor
mvn -pl module_name在指定模块上执行命令
mvn -ff 在递归执行命令过程中，一旦发生错误就直接退出
mvn -Dxxx=yyy指定java全局属性
mvn -Pxxx引用profile xxx
```

- Lifecycle 中的命令
```
mvn test-compile 编译测试代码
mvn test 运行程序中的单元测试
mvn compile 编译项目
mvn package 打包，此时target目录下会出现maven-quickstart-1.0-SNAPSHOT.jar文件，即为打包后文件
mvn install 打包并安装到本地仓库，此时本机仓库会新增maven-quickstart-1.0-SNAPSHOT.jar文件。
每个phase都可以作为goal，也可以联合，如之前介绍的mvn clean install
```

- 其他命令
```
mvn archetype:generate 创建maven项目
mvn package 打包，上面已经介绍过了
mvn package -Prelease打包，并生成部署用的包，比如deploy/*.tgz
mvn install 打包并安装到本地库
mvn eclipse:eclipse 生成eclipse项目文件
mvn eclipse:clean 清除eclipse项目文件
mvn site 生成项目相关信息的网站
```

## 参考链接
1. <http://www.oracle.com/technetwork/cn/community/java/apache-maven-getting-started-1-406235-zhs.html>
2. <http://www.oracle.com/technetwork/cn/community/java/apache-maven-getting-started-2-405568-zhs.html>
3. <http://www.trinea.cn/android/maven/>
4. <http://maven.apache.org/guides/index.html>
