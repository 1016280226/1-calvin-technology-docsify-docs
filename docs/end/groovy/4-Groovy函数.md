# 笔记四 Groovy 函数

## 1. Groovy 函数使用

- **`def`** 定义函数**关键字**

- 函数定义写法：

  ```groovy
  def printMsg (msg) {
  	println(msg)
  	// 执行内容
  	return msg;
  }
  ```

- 示例代码：

  ```groovy
  // 编写一个打印信息的函数
  def printMsg(msg){
      println(msg)
  }
  // 编写一个加法的函数
  def addMath(a,b){
     return a + b;
  }
  
  // 调用打印信息的函数
  printMsg("你好！Groovy 语言")
  
  // 调用打印信息的函数
  printMsg("1 + 1 = " + addMath(1,1))
  ```

  <font color="green"><b>输出结果:</b></font>

  <img src="./statics/images/groovy/groovy_funtion_define.png" style="zoom:100%;" />

​    

## 2. Groovy 正则表达式

- 写法示例：

  ```groovy
  // 该注解json 序列化问题
  //@NonCPS
  String getBranch(String branchName) {
      def matcher = (branchName =~ "RELEASE-[0-9]{4}")
      if (matcher.find()){
          newBranchName = matcher[0]
      } else {
          newBranchName = branchName
      }
      return newBranchName
  }
  
  branchName = "1.4.6.RELEASE"
  newBranchName = getBranch(branchName)
  println("新的分支名 ---> ${newBranchName}")
  ```

  <font color="green"><b>输出结果:</b></font>

  <img src="./statics/images/groovy/groovy_regpatten_rule.png" style="zoom:100%;" />

  

## 3. Groovy 构建打包示例

- 代码示例：

  ```groovy
  // 构建打包
  def Build(javaVersion,buildType,buildDir,buildShell){
      if (buildType == 'maven'){
          Home = tool '/usr/local/apache-maven'
          buildHome = "${Home}/bin/mvn"
      } else if (buildType == 'ant'){
          Home = tool 'ANT'
          buildHome = "${Home}/bin/ant"
      } else if (buildType == 'gradle'){
          buildHome = '/usr/local/bin/gradle'
      } else{
          error 'buildType Error [maven|ant|gradle]'
      }
      echo "BUILD_HOME: ${buildHome}"
      
      // 选择JDK版本
      jdkPath = ['jdk7' : '/usr/local/jdk1.7.0_79',
                 'jdk6' : '/usr/local/jdk1.6.0_45',
                 'jdk8' : '/usr/java/jdk1.8.0_111',
                 'jdk11': '/usr/local/jdk-11.0.1',
                 'null' : '/usr/java/jdk1.8.0_111']
      def javaHome = jdkPath["$javaVersion"]
      if ("$javaVersion" == 'jdk11'){
         sh  """
          export JAVA_HOME=${javaHome}
          export PATH=\$JAVA_HOME/bin:\$PATH
          java -version
          cd ${buildDir} && ${buildHome} ${buildShell}
          """
      } else {
          sh  """
              export JAVA_HOME=${javaHome}
              export PATH=\$JAVA_HOME/bin:\$PATH
              export CLASSPATH=.:\$JAVA_HOME/lib/dt.jar:\$JAVA_HOME/lib/tools.jar
              java -version
              cd ${buildDir} && ${buildHome} ${buildShell}
              """
      }
  
  }
  ```

  ```groovy
      
   //前端Build
  def WebBuild(srcDir,serviceName){
      def deployPath = "/srv/salt/${JOB_NAME}"
      sh """
          [ -d ${deployPath} ] || mkdir -p ${deployPath}
          cd ${srcDir}/
          rm -fr *.tar.gz 
          tar zcf ${serviceName}.tar.gz * 
          cp ${serviceName}.tar.gz ${deployPath}/${serviceName}.tar.gz
          cd -
      """
  }
  ```

  > *该示例引用网站：https://github.com/zeyangli/ShareLibrary-jenkins/blob/master/src/org/devops/build.groovy*