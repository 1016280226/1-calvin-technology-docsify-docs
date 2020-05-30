# 笔记一 Groovy 简介

<img src="./statics/images/groovy/groovy_introduction_01.png" style="zoom:100%;"/>

## 1. 为什么学习 Groovy 语言?

> 因为编写 **`Pipeline`** 流水线使用的语法是 **`Groovy`**, 同时它也支持 **Java** 平台，无论是后台开发还是运维人员学习 **`Groovy`** 语言是一门不错选择。
>
> 官方网址：http://groovy-lang.org



## 2. Groovy 特性

- **`Groovy`** 是一种**功能强大**、**可选类型**和**动态**语言, 支持 **Java** 平台。

- **语法简洁**，熟悉且简单易学。

- 与 **Java** 程序顺利集成，并立即为您的应用程序提供强大的功能，包括**脚本编写**功能，**特定领域语言**编写，运行时和编译时**元编程**以及**函数式**编程。

  

<img src="./statics/images/groovy/groovy_introduction.png" style="zoom:100%;"/>



## 3. Groovy 安装

1. 先安装 **JDK** , 设置 **JDK** 环境变量。

2. 下载安装包 **apache-groovy-sdk-3.0.4.zip**。

3. 解压安装包。

   ```bash
   $ unzip apache-groovy-sdk-3.0.4.zip
   ```

4. 获取到安装包 **bin** 目录，设置（**Linux**）环境变量。

   ```bash
   # 编辑添加环境变量
   $ vim /etc/profile
   ------------------------- 添加一下内容 --------------------------------
   export GROOVY_HOME= xxx(解压后安装包目录)
   export PATH=$PATH:$GROOVY_HOME/bin
   ---------------------------------------------------------------------
   # 让它生效
   $ source /etc/profile
   ```

5. 使用命令 **groovyconsole** 打开 **Groovy 编译器**，可以看到弹出一个 **Groovy 编译器**

   ```bash
   $ groovyconsole
   ```

   <img src="./statics/images/groovy/groovy_vm_01.png" style="zoom:100%;" />

6. 也可以使用命令 **groovysh** 进行命令行编译

   ```bash
   $ groovysh
   ```

   

## 4. Groovy 使用

1. 编写一个 打印“**Hello World**” ，保存**HelloWord .groovy** 文件。

   ```groovy
   println("Hello Word !")
   ```

   <font color=green><b>输出以下内容:</b></font>

   <img src="./statics/images/groovy/groovy_helloword_01.png" style="zoom:100%;" />

2. 或者，在控制台中使用命令方式执行。

   ```bash
   $ groovy HelloWord.groovy
   ```

   