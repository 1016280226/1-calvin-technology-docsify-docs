



# 笔记二  Pipeline 语法



## 1. 声明式流水线

声明式 **`Pipleine`** 是官方推荐的语法，声明式语法更加简洁。所有的声明式 **`Pipeline`** 都必须包含一个 **`pipeline`** 块中。

在声明式 **Pipeline** 中的基本语句和表达式遵循 **Groovy** 的语法。

示例：

```groovy
pipeline {
    //run
}
```

> *****
>
> *Warnning 以下例外：*
>
> - 流水线顶层必须是一个块，特别是 **`pipeline{}`**。
> - 不需要分号作为分割符，是按照行分割的。
> - 语句块只能由阶段、指令、步骤、赋值语句组成。例如: input 被视为 input()。



## 2. agent (代理) 

**Jenkins** 通过将分布式构建委托给“**代理/agent**”节点来提供执行分布式构建的能力。 这样做可以使您仅使用 Jenkins 服务器的一个实例来执行多个项目，而工作负载却被分配给了它的代理 。

**`agent`**：指定了流水线的执行节点。

**`agent`** 参数如下：

<style>
    .table11_3 table {
        width:100%;
        margin:15px 0;
        border:3px;
    }
    .table11_3 th {
        background-color:#46D77B;
        color:#706D75
    }
    .table11_3,.table11_3 th,.table11_3 td {
        font-size:1em;
        text-align:center;
        padding:14px;
        border-collapse:collapse;
    }
    .table11_3 th {
        border: 1px solid #fe2047;
        border-width:1px 0 1px 0;
        border:1px inset #ffffff;
    }
    .table11_3 td {
        text-align:left;
        border: 1px solid #fe2047;
        border-width:1px 0 1px 0;
        border:1px inset #ffffff;
    }
    .table11_3 tr {
        border: 1px solid #ffffff;
    }
    .table11_3 tr:nth-child(odd){
        background-color:#F3F5F5;
    }
    .table11_3 tr:nth-child(even){
        background-color:#ffffff;
    }
</style>
<table class=table11_3>
    <tr>
     <th>参数</th><th>含义</th>
    </tr>
    <tr>
      <td><b>any</b></td>
      <td>在任何可用的节点上执行流水线。</td>
    </tr>
    <tr>
      <td><b>none</b></td>
      <td>没有指定agent的时候默认。</td>
    </tr>
    <tr>
      <td><b>lable</b></td>
      <td>在指定标签上的节点上运行流水线。</td>
    </tr>
    <tr>
      <td><b>node</td>
      <td> 允许额外的选项。 </td>
    </tr>
</table>


示例：

``` groovy
pipeline {
    // 意味着 Jenkins 将在任何可用节点上运行任务
    agent any
}
```

```groovy
pipeline {
   // 没有指定节点，默认自己去寻找节点，随机分配
   agent none
}
```

以下两种写法意思一样

```groovy
pipeline {
    agent {
        // 在指定标签上的节点上运行流水线。
        node {
            label 'labelname'
        }
    }
}
```

```groovy
pipeline {
    agent {
        // 在指定标签上的节点上运行流水线。
        label 'labelname'
    }        
}  
```



## 3. stages (阶段)

**`stages`** 包含一系列一个或多个 **stage** 指令，建议 **stages** 至少包含一个 **stage** 指令用于连续交付过程的每个离散部分，比如：构建、测试、部署。

示例：

```groovy
pipeline {
    agent any
    stages ('stages'){
      // 阶段1 获取代码
      stage('CheckOut') {
         steps {
            script {
               println("获取代码")
            }
         }
      }

      // 阶段2 构建
      stage("Build") {
        steps {
            script {
               println("运行构建")
            }
        }
      }
        
      // 阶段3 测试
      stage("Test") {
        steps {
            script {
               println("运行测试")
            }
        }
      }
       
      // 阶段4 部署发布
      stage("sh") {
        steps {
            script {
               println("部署发布")
            }
        }
      }  
    }
}
```



## 3. steps (步骤)

**`steps`** 是每一个(**stages**)阶段中要执行的每一个步骤。

示例：

```groovy
pipeline {
    agent any
    stages ('Example'){
        // 执行的步骤
        steps {
            echo 'Hello World'
        }
    }
}
```



## 4. environment (环境变量)

**`environment`** 指令指定一个键值对序列，该序列将被定义为所有步骤的环境变量，或者特定于阶段的步骤，这取决于 **environment** 指令在流水线内的位置。

**environment**该指令支持一个特殊的方法 **credentials()**, 该方法可用于在**jenkins** 环境中通过标识符访问预定义的凭证。

- 对于类型为“**Secret Text**” 的凭证，**credentials**() 将确保指定的环境变量包含秘密文本内容。
- 对于类型为“**SStandard username and password**”的凭证，指定环境变量指定为**username:password**,并且两个额外的环境变量将被自动定义: 分别为 **MYVARNAME_USER** 和 **MYVRNAME_PSW**

示例：

```groovy
pipeline {
   agent any
   // 全局环境变量 
   environment {
        J = 'java'
   }
    stages ('Environment Example') {
        steps {
            // 引用全局变量
            echo "${J}"
        }
    }
}
```

```groovy
pipeline {
   agent any
   stages ('Part Environment Example') {
      // 局部环境变量
      environment {
         AN_ACCESS_KEY = credentials('my-prefined-secret-text') 
      }
      steps {
       	 // 引用局部变量
         echo "${AN_ACCESS_KEY}"
      }
   }
}
```



## 5. options (选项)

**`options`** 指令允许从流水线内部配置特定于流水线的选项。

- 流水线提供了许多这样的选项,比如：**buildDiscarder** 。

- 但也可以由插件提供，比如：**timestamps**。

**`options`** 选项其他功能, 如下：

<style>
    .table11_3 table {
        width:100%;
        margin:15px 0;
        border:3px;
    }
    .table11_3 th {
        background-color:#46D77B;
        color:#706D75
    }
    .table11_3,.table11_3 th,.table11_3 td {
        font-size:1em;
        text-align:center;
        padding:14px;
        border-collapse:collapse;
    }
    .table11_3 th {
        border: 1px solid #fe2047;
        border-width:1px 0 1px 0;
        border:1px inset #ffffff;
    }
    .table11_3 td {
        text-align:left;
        border: 1px solid #fe2047;
        border-width:1px 0 1px 0;
        border:1px inset #ffffff;
    }
    .table11_3 tr {
        border: 1px solid #ffffff;
    }
    .table11_3 tr:nth-child(odd){
        background-color:#F3F5F5;
    }
    .table11_3 tr:nth-child(even){
        background-color:#ffffff;
    }
</style>
<table class=table11_3>
    <tr>
     <th>参数</th><th>含义</th>
    </tr>
    <tr>
      <td><b>buildDiscarder</b></td>
      <td>为最近的流水线运行的特定数量保存组件和控制台输出</td>
    </tr>
    <tr>
      <td><b>disableConcurrentBuilds</b></td>
      <td>不允许同时执行流水线。可被用来防止同时访问共享资源。</td>
    </tr>
    <tr>
      <td><b>overrideIndexTriggers</b></td>
      <td>允许覆盖分支索引的默认处理。</td>
    </tr>
    <tr>
      <td><b>skipDefaultCheckout</td>
      <td> 在 agent 指令中，跳过从源代码控制中检出代码的默认情况。 </td>
    </tr>
    <tr>
      <td><b>skipStagesAfterUnstable</b></td>
      <td> 一旦构建状态变得 <b>UNSTABLE</b>, 跳过该阶段。</td>
    </tr>
    <tr>
      <td><b>checkoutToSubdirectory</b></td>
      <td>在工作空间的子目录中自动执行源代码控制检出。</td>
    </tr>
    <tr>
      <td><b>timeout</b></td>
      <td>设置流水线运行的超时时间，在此之后<b>Jenkins</b>将中止流水线。</td>
    </tr>
    <tr>
      <td><b>retry</b></td>
      <td>在失败之后，重试整改流水线指定的次数。</td>
    </tr>
   <tr>
      <td><b>timestamps</b></td>
      <td>预测所有的流水线生成的控制台输出，与该流水线发出时间一致。 </td>
    </tr> 
</table>

示例：

```groovy
// 指定【一个小时】的全局执行超时，在此之后 Jenkins 将【终止】流水线运行。
pipeline {
    agent any
    
    // 设置选项：执行超时时间为1个小时
    options {
        timeout(time:1, unit: 'HOURS')
    }
    
    stages {
        stage {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```



## 6. paramtes (参数)

**paramtes** 为流水运行时设置项目相关的参数：

<style>
    .table11_3 table {
        width:100%;
        margin:15px 0;
        border:3px;
    }
    .table11_3 th {
        background-color:#46D77B;
        color:#706D75
    }
    .table11_3,.table11_3 th,.table11_3 td {
        font-size:1em;
        text-align:center;
        padding:14px;
        border-collapse:collapse;
    }
    .table11_3 th {
        border: 1px solid #fe2047;
        border-width:1px 0 1px 0;
        border:1px inset #ffffff;
    }
    .table11_3 td {
        text-align:left;
        border: 1px solid #fe2047;
        border-width:1px 0 1px 0;
        border:1px inset #ffffff;
    }
    .table11_3 tr {
        border: 1px solid #ffffff;
    }
    .table11_3 tr:nth-child(odd){
        background-color:#F3F5F5;
    }
    .table11_3 tr:nth-child(even){
        background-color:#ffffff;
    }
</style>
<table class=table11_3>
    <tr>
     <th>类型</th>
     <th>含义</th>
    </tr>
    <tr>
      <td><b>string</b></td>
      <td>字符串类型的参数</td>
    </tr>
    <tr>
      <td><b>booleanParam</b></td>
      <td>布尔类型参数</td>
    </tr>
</table>

示例：

```groovy
parameters {
	string(name: 'DEPLOY_ENV', defaultValue: 'staging', description: '')
}
```

```groovy
parameters {
	booleanParam(name: 'DEBUG_BUILD', defaultValue: true, description: '')
}
```





> *Warnning:*
>
> - **自定义**：*在流水线中参数必须执行一遍后才有参数变量（初始化过程）。*
>
> - **UI**: *界面定义参数不需要初始化。*



## 7. post (构建后操作)

**`post`** 在 **Stage** 运行阶段完成情况，所产生后果（不同状态），导致 **post** 运行不同构建后操作。

**`post`** 构建后操作运行参数如下：

<style>
    .table11_3 table {
        width:100%;
        margin:15px 0;
        border:3px;
    }
    .table11_3 th {
        background-color:#46D77B;
        color:#706D75
    }
    .table11_3,.table11_3 th,.table11_3 td {
        font-size:1em;
        text-align:center;
        padding:14px;
        border-collapse:collapse;
    }
    .table11_3 th {
        border: 1px solid #fe2047;
        border-width:1px 0 1px 0;
        border:1px inset #ffffff;
    }
    .table11_3 td {
        text-align:left;
        border: 1px solid #fe2047;
        border-width:1px 0 1px 0;
        border:1px inset #ffffff;
    }
    .table11_3 tr {
        border: 1px solid #ffffff;
    }
    .table11_3 tr:nth-child(odd){
        background-color:#F3F5F5;
    }
    .table11_3 tr:nth-child(even){
        background-color:#ffffff;
    }
</style>
<table class=table11_3>
    <tr>
     <th>参数</th>
     <th>含义</th>
     <th>例如</th>
    </tr>
    <tr>
      <td><b>always</b></td>
      <td>总是，无论流水线或者阶段的什么状态或产生后果，它都会执行。</td>
      <td></td>
    </tr>
    <tr>
      <td><b>changed</b></td>
      <td>变更，和之间完成状态或产生后果不同，才会执行。</td>
      <td></td>
    </tr>
    <tr>
      <td><b>failure</b></td>
      <td>失败，当流水线或阶段执状态为“<b>failure</b>”, 才执行。</td>
      <td>运行 <b>stages 阶段</b>报错或执行不下去。</td>
    </tr>
    <tr>
      <td><b>success </td>
      <td>成功，当流水线或阶段执状态为“<b>success</b>”, 才执行。</td>
      <td>运行 <b>stages 阶段</b>顺利执行下去。</td>
    </tr>
    <tr>
      <td><b>unstable </td>
      <td>不稳定，当流水线或阶段执状态为“<b>unstable</b>”, 才执行。</td>
      <td>测试失败。</td>
    </tr>
     <tr>
      <td><b>aborted </td>
      <td>取消，当流水线或阶段执状态为“<b>aborted</b>”, 才执行。</td>
      <td>手动取消构建。</td>
    </tr>
</table>

示例：

```groovy
pipeline {
   agent any
    
   stages('Test') {
       steps {
           echo 'Test'
       }    
   } 
    
   // 根据流水线状态做构建后的操作
   post {
      always {
	     script {
			println("流水线结束后，经常做的事情")
		 }
	  }
	  
	  success {
		 script {
			println("流水线成功后, 要做的事情")
		 }
	  }
	  
	  failure {
		 script {
		   println("流水线失败后, 要做的事情")
		 }
	  }
	  
	  aborted {
		 script {
		   println("流水线取消后, 要做的事情")
		 } 	
	  } 
   } 
} 
```



## 8. trigger (触发器)

**`trigger`** 构建触发器。

**`trigger`** 触发器类型，如下：

<style>
    .table11_3 table {
        width:100%;
        margin:15px 0;
        border:3px;
    }
    .table11_3 th {
        background-color:#46D77B;
        color:#706D75
    }
    .table11_3,.table11_3 th,.table11_3 td {
        font-size:1em;
        text-align:center;
        padding:14px;
        border-collapse:collapse;
    }
    .table11_3 th {
        border: 1px solid #fe2047;
        border-width:1px 0 1px 0;
        border:1px inset #ffffff;
    }
    .table11_3 td {
        text-align:left;
        border: 1px solid #fe2047;
        border-width:1px 0 1px 0;
        border:1px inset #ffffff;
    }
    .table11_3 tr {
        border: 1px solid #ffffff;
    }
    .table11_3 tr:nth-child(odd){
        background-color:#F3F5F5;
    }
    .table11_3 tr:nth-child(even){
        background-color:#ffffff;
    }
</style>
<table class=table11_3>
    <tr>
     <th>类型</th>
     <th>含义</th>
    </tr>
    <tr>
      <td><b>cron</b></td>
      <td>计划任务定期执行构建。</td>
    </tr>
    <tr>
      <td><b>pollSCM</b></td>
      <td>与<b>cron</b>定义类似，作用在<b>Jenkins</b>定期检测源码变化。</td>
    </tr>
    <tr>
      <td><b>upstream</b></td>
      <td>接受逗号分隔的工作字符串或阈值。当字符串中的任何作业以最小阈值结束时，流水线被重新触发。</td>
    </tr>
</table>

示例：

```groovy
triggers {
	cron('H */4 * * 1-5')
}
```

```groovy
triggers {
	pollSCM('H */4 * * 1-5')
}
```

```groovy
triggers {
	upstream(upstreamProjects: 'job1,job2', threshold: hudson.model.Result.SUCCESS)
}
```



## 9. tool（工具）

**`tool`** 获取通过自动安装或手动放置工具的环境变量。

**`tool`**支持**maven/jdk/gradle** 的环境变量。

工具的名称必须在系统设置 -> **全局工具配置中定以**。

示例：

```groovy
pipeline {
	agent any
    
    // 从全局工具配置中的名称
    tools {
        maven 'apache-maven-3.6.3'
        jdk 'openjdk-8'
    }
    
    stages {
        stage ('Example') {
            steps {
                // 执行查看 Maven 版本
                sh 'mvn -v'
                // 执行查看 jdk 斑斑
                sh 'java -version'
            }
        }
    }
}
```



## 10. input (输入)

**`input`** 用户在执行各个阶段的时候，由人工确认是否继续进行。

**`input`** 参数说明, 如下：

<style>
    .table11_3 table {
        width:100%;
        margin:15px 0;
        border:3px;
    }
    .table11_3 th {
        background-color:#46D77B;
        color:#706D75
    }
    .table11_3,.table11_3 th,.table11_3 td {
        font-size:1em;
        text-align:center;
        padding:14px;
        border-collapse:collapse;
    }
    .table11_3 th {
        border: 1px solid #fe2047;
        border-width:1px 0 1px 0;
        border:1px inset #ffffff;
    }
    .table11_3 td {
        text-align:left;
        border: 1px solid #fe2047;
        border-width:1px 0 1px 0;
        border:1px inset #ffffff;
    }
    .table11_3 tr {
        border: 1px solid #ffffff;
    }
    .table11_3 tr:nth-child(odd){
        background-color:#F3F5F5;
    }
    .table11_3 tr:nth-child(even){
        background-color:#ffffff;
    }
</style>
<table class=table11_3>
    <tr>
     <th>参数</th>
     <th>含义</th>
    </tr>
    <tr>
      <td><b>message</b></td>
      <td>呈现给用户的提示信息。</td>
    </tr>
    <tr>
      <td><b>id</b></td>
      <td>可选，默认为<b>stage</b>名称。</td>
    </tr>
    <tr>
      <td><b>ok</b></td>
      <td>默认表单上的ok文本。</td>
    </tr>
    <tr>
      <td><b>parameters</b></td>
      <td>提示提交者提供一个可选参数列表。</td>
    </tr>
    <tr>
      <td><b>submitter</b></td>
      <td>可选的，以逗号分隔的用户列表或允许提交的外部组名。默认，允许任何用户。</td>
    </tr>
</table>

示例：

```groovy
pipeline {
	agent any
    stages {
        stage ('Input Test') {
            steps {  
                input {
                	id: 'Test', 
                	message: '你很帅吗？', 
                	ok: '是的，有一点点', 
                	parameters: [choice(choices: ['你是帅的', '你是丑的'], description: '', name: 'test1')], 
                	submitter: 'Administrator'
                }
            }
        }
    }
}
```



## 11. when (判断)

**`when`** 指令允许流水线根据给定的条件是否应该执行的阶段。

**`when`** 指令必须包含一个条件。

如果 **`when`** 指令包含多个条件，所有的子条件必须返回 **true**， 阶段才能执行。

这与子条件在**allOf** 条件下嵌套的情况相同。

**内置条件**参数，如下：

<style>
    .table11_3 table {
        width:100%;
        margin:15px 0;
        border:3px;
    }
    .table11_3 th {
        background-color:#46D77B;
        color:#706D75
    }
    .table11_3,.table11_3 th,.table11_3 td {
        font-size:1em;
        text-align:center;
        padding:14px;
        border-collapse:collapse;
    }
    .table11_3 th {
        border: 1px solid #fe2047;
        border-width:1px 0 1px 0;
        border:1px inset #ffffff;
    }
    .table11_3 td {
        text-align:left;
        border: 1px solid #fe2047;
        border-width:1px 0 1px 0;
        border:1px inset #ffffff;
    }
    .table11_3 tr {
        border: 1px solid #ffffff;
    }
    .table11_3 tr:nth-child(odd){
        background-color:#F3F5F5;
    }
    .table11_3 tr:nth-child(even){
        background-color:#ffffff;
    }
</style>
<table class=table11_3>
    <tr>
     <th>参数</th>
     <th>含义</th>
    </tr>
    <tr>
      <td><b>branch</b></td>
      <td>当正在构建的分支与模式给定的<b>分支匹配时</b>，执行这个阶段，这适用于多分支流水线。</td>
    </tr>
    <tr>
      <td><b>environment</b></td>
      <td>当环境变量<b>给定的值时</b>，执行这个步骤。</td>
    </tr>
    <tr>
      <td><b>expression</b></td>
      <td>当指定的<b>Groovy</b>表达式评估为<b>true时</b>，执行这个步骤。</td>
    </tr>
    <tr>
      <td><b>not</b></td>
      <td>当嵌套条件<b>错误时</b>，执行这个阶段，必须包含一个条件。</td>
    </tr>
    <tr>
      <td><b>allOf</b></td>
      <td>当所有的嵌套条件<b>都正确时</b>，执行这个阶段，必须包含至少一个条件。</td>
    </tr>
     <tr>
      <td><b>anyOf</b></td>
      <td>当至少有一个嵌套条件为<b>true时</b>，执行这个阶段，必须包含至少一个条件。</td>
    </tr>
</table>

示例：

```groovy
when {
    // 1.branch 当正在构建的分支与模式给定的分支匹配时，执行这个阶段，这适用于多分支流水线。
	branch 'master'
}
```

```groovy
when {
    // 2.当环境变量给定的值时，执行这个步骤。
    environment name: 'DEPLOY_TO', value: 'production'
}
```

```groovy
when {
    // 3.expression 当指定的Groovy表达式评估为true时，执行这个步骤。
	expression {
		return params.DEBUG_BUILD
	}
}
```

```groovy
when {
    // 4.not 当嵌套条件错误时，执行这个阶段，必须包含一个条件。
    not {
        branch 'master'
    }
}
```

```groovy
when {
    // 5.allOf 当所有的嵌套条件都正确时，执行这个阶段，必须包含至少一个条件。
    allOf {
        branch 'master';
        environment name: 'DEPLOY_TO', value: 'production'
    }
}
```

```groovy
when {
    // 6.当至少有一个嵌套条件为true时，执行这个阶段，必须包含至少一个条件。
	anyOf {
	  	branch 'master';
	  	branch 'staging'
	}
}
```

```groovy
// 完整案例
pipeline {
	agent any
    
    stages {
        stage ('When Example') {
            when {
                environment name: 'isSkip When Example', value: 'true'
            }
            steps {
                println("When Example Execute Success")    
            }
        }
        stage ('Input Example') {
            steps {
                input id: 'Test', 
                message: '你很帅吗？', 
                ok: '是的，有一点点', 
                parameters: [choice(choices: ['你是帅的', '你是丑的'], description: '', name: 'test1')], 
                submitter: 'Administrator'
            }
        }
    }
}
```



## 12. paraller (并行)

**`parller`** 声明式流水线的**阶段**可以在他们的**内部声明多隔嵌套阶段**，它们将并行执行。

> *Warnning ：*
>
> - 一个阶段必须只有一个 **steps** 或 **parallel** 的阶段。
> - 嵌套本身不能包含进一步的 **parallel** 阶段， 但是其他的阶段的行为与任何其他**stage** **parallel** 的阶段不能包含 **agent** 或 **tools** 阶段，因为没有相关 **steps**。
> - 另外， 通过添加 **failFast** **true** 到包含 **parallel** 的 **stage** 中， 当其中一个进程失败时，你可以强制所有的 **parallel** 阶段都被终止。

示例：

```groovy
pipeline {
   agent any
   stages {
      stage('CheckOut GitLab Code') {
         steps {
            script {
               println("获取代码")
            }
         }
      }

      stage("Build") {
		failFast true
		// 并行
		parallel {
			stage("Scan GitLab Code") {
				steps {
					println("代码扫描")
				}
			}
			stage("Maven Build") {
				steps {
					println("Maven 打包构建")
				}
			} 
		} 
      }
   }
}
```



## 13. script (脚本)

**`script`** 步骤需要 **[scripted-pipeline]**块并在声明式流水线中执行。

对于大多数用例来说，应该声明式流水线中的“**脚本**”步骤是不必要的，但是它可以提供一个有用的“**逃生出口**”。

非平凡的规模或复杂性的**`script`**块应该被转移到**共享库**。

示例：

```groovy
pipeline {
	agent any
    stages {
        stage('Script Example') {
            echo 'Hello World'
        }
        script {
            def browsers = ['chrome', 'firefox']
            for (int i = 0; i < browsers.size(); ++i) {
                echo "Testing the ${browsers[i] browser}"
            }
        }
    }
}
```



## 14. Pipeline 基本语法汇总演示

主要编写了一个简单 **`Jenkinsfile`** 文件，流程包含了开发人员开发到发布。

有以下过程：**拉取代码 -> 代码扫描 -> 打包构建 -> 发布部署**。

代码示例如下：

```groovy
pipeline {
   // 1.agent 指定运行流水线的节点, any 代码任务节点
   agent any
   // 5.environment 配置全局环境变量
   environment {
	  GITLAB_ROOT_ACCESS_KEY = credentials('pipeline-gitlab-test-secret-text')
   }
   // 6.options 设置超时
   options {
	  timeout(time:1, unit: 'HOURS')
   }
   // 7.parameters 设置构建环境为 UAT
   parameters {
	  string(name: 'BUILD_ENV', defaultValue: 'UAT', description: '')
   }
   // 8.tool 工具使用
   tools {
	  jdk 'openjdk-8'
   }
   // 2.stages 流水线的阶段
   stages {
      // 3.stage 阶段1: 获取代码
      stage('CheckOut GitLab Code') { 
		 // 4.steps 步骤打印获取代码
         steps {
            script {
			   println("设置流水线超时时间为：1个小时")
               println("阶段1：使用密钥: ${GITLAB_ROOT_ACCESS_KEY}, 获取【GitLab】代码")
            }
         }
      }
      stage("Build") {
		failFast true
		// 12. parallel 并行操作
		parallel {
			stage("Parallel Ops") {
			   steps {
				  println("阶段2: 并行操作")
			   }
			}
			stage("Scan GitLab Code") {
			   steps {
				  println("阶段2.1: 代码扫描")
			   }
			}
			stage("Maven Build") {
				// 11.when 判断运行环境是否为UAT
				when {
				   environment name: 'BUILD_ENV', value: 'UAT'
				}	
				steps {
				   // 10.input 输入个阶段
				   input message: "是否需要 Maven 打包构建？", ok: '是'
				   println("阶段2.2: Maven打包构建")
				   println("构建环境为：${BUILD_ENV}")
				   println("使用Jdk 版本:")
				   sh 'java -version'
				}
			} 	
		} 
      }
	  stage("Deployment") {
		 steps {
			println("阶段3：发布部署")
		 }	 
	  }
   }
   // 8.POST 根据流水线状态做一些任务
   post {
      always {
         script {
            println("流水线结束后，经常做的事情")
         }
      }
      success {
         script {
            println("流水线执行成功啦, YES, SO GO! 骚 Easy! ")
         }
      }
      failure {
         script {
           println("流水线执行失败啦, 想哭")
         }
      }
      aborted {
         script {
           println("流水线取消后, WHAT Happen TO YOU ? WHAT DO YOU DO!")
         }     
      } 
   }
}
```

### 第一次构建：参数变量没有被初始化，所以在 **`when {}`** 代码块中被跳过。

<img src="./statics/images/jenkins/pipeline/pipeline_jenkinsfile_syntax_01.gif" style="zoom:200%;"/>

### 第二次构建：执行的完整的流程。

<img src="./statics/images/jenkins/pipeline/pipeline_jenkinsfile_syntax_02.gif" style="zoom:200%;"/>

