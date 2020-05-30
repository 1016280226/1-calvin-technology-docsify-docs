# 笔记三 Groovy 条件与循环语句

## 1. <font color="#ff6702"><b>if</b></font> 条件语句

- **`if`** 语句写法：

  ```groovy
  if (表达式1) {
      // 执行内容
  } else if (表达式2) {
      // 执行内容
  } else {
      // 执行内容
  }
  ```

- 示例代码：

  ```groovy
  envType = "DEV"
  if (envType == "DEV") {
      println("这个是开发环境")
  } else if (envType == "UAT") {
      println("这个是测试环境")
  } else {
     println("这个是生产环境")
  }
  ```

  <font color="green"><b>输出结果:</b></font>

  <img src="./statics/images/groovy/groovy_if_syntax.png" style="zoom:100%;" />

  

## 2. <font color="#ff6702"><b>switch</b></font> 条件语句

- **`switch`** 语句写法：

  ```groovy
  switch ("枚举、类型判断、数字") {
  	case "枚举.生产环境"：
  	// 执行内容
  	break;
  	case "枚举.测试环境"：
  	// 执行内容
  	break;
  	case "枚举.开发环境"：
  	// 执行内容
  	break;
  	default:
  	// 执行内容
  }
  ```

  示例代码：
  
  ```groovy
  envType = "UAT"
  switch ("${envType}") {
      case "DEV":
          println("这个是开发环境")
          break;
      case "UAT":
          println("这个是测试环境")
          break;
      default:
          println("这个是生产环境")
  }
  ```
  
  <font color="green"><b>输出结果:</b></font>
  
  <img src="./statics/images/groovy/groovy_switch_syntax.png" style="zoom:100%;" />
  
  

## 3. <font color="#ff6702"><b>for/while</b></font> 条件语句

- **`for`** 语句写法：

  ```groovy
  for (i in list) {
     // 执行内容
  }
  ```
  
  示例代码：
  
  ```groovy
  langs = ['java', 'python', 'groovy']
  for (lang in langs) {
      println("lang is ${lang}")
  }
  ```
  
  <font color="green"><b>输出结果:</b></font>
  
  <img src="./statics/images/groovy/groovy_for_syntax.png" style="zoom:100%;" />
  
  
  
- **`while`** 语句写法：

  ```groovy
  while ((表达式为true)) {
      // 执行内容
  }
  ```

  示例代码：

  ```groovy
  i = 1
  while (i < 100) {
      println("i ："+ i )
      i++
  }
  ```

  <font color="green"><b>输出结果:</b></font>

  <img src="./statics/images/groovy/groovy_while_syntax.png" style="zoom:100%;" />

