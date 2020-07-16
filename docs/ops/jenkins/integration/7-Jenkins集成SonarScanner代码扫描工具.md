# 笔记三 Jenkins 集成SonarScanner 代码扫描工具



## 1. SonnarScanner 是什么 ？

> 参考 :[SonarQuber 简介](./docs/ops/sonarqube/1-SonarQube简介.md)和 [SonarQuber 安装和使用](./docs/ops/sonarqube/2-SonarQube安装和使用.md)

## 2. Jenkins 集成 SonnarScanner 步骤

### 2.1 安装 <font color=#ff6702>SonanerQube Scanner for Jenkins</font> 插件

- 点击“**`可选插件`**”
- 搜索框中输入：**`SonanerQube Scanner for Jenkins`**
- 下载安装重启

<img src="./../../../statics/images/jenkins/integration/jenkins_sonarscanner_plugs_download.png" alt="image-20200716155600616" style="zoom:100%;" />

### 2.2 在 <font color=#ff6702>全局工具配置</font>设置<font color=#ff6702>SonarQube Scanner</font>

- Name : **`SonarQube`**
- 勾选: **`自动安装`**
- 选择 **Install from Maven Central**: **`SonarQuber Scanner 4.3.0.2.102`**
- 保存或应用

<img src="./../../../statics/images/jenkins/integration/jenkins_sonarquber_scanner_tool_config.png" style="zoom:100%;" />

### 2.3 在 Jenkins<font color=#ff6702>系统配置</font> 添加 <font color=#ff6702>SonarQube Servers</font>

- 勾选: **`Enable injection of SonarQube server configuration as build environment variables`** （启用将SonarQube服务器配置注入为构建环境变量）
- Name: **`SonarQube-Test-Server`**
- Server URL：**`http://192.168.2.167`**

- Server authentication token ：**`sonarqube-admin`**

  - 使用 **`admin`** 账号登录 **`SonarQube Server`** ，在右上方点击有一个“**`我的账号`**”。

    <img src="./../../../statics/images/jenkins/integration/jenkins_sonarqube_server_my_account.png" alt="image-20200716161443290" style="zoom:100%;" />

  - 点击“**`安全`**”，生成令牌输入“**`jenkins`**”,点击“**`生成`**”。

    <img src="./../../../statics/images/jenkins/integration/jenkins_integration_admin_account_gen.png" alt="image-20200716161835183" style="zoom:100%;" />

  - 拷贝密钥到 Jenkins : **`33794503437d4832062a43478c49cc150f6164ab`**

    <img src="./../../../statics/images/jenkins/integration/jekins_sonarqube-admin_account_add.png" alt="image-20200716162141301" style="zoom:100%;" />

  - 最后点击**`保存`**或**`应用`**。
  
    <img src="./../../../statics/images/jenkins/integration/jenskins_sonarqube_servers_config.png" alt="image-20200716161158080" style="zoom:100%;" />


### 2.4 在 GitLab 中 Jenkins-share-library 项目共享库添加 SonarQube 代码扫描工具类

- 在 **`Jenkins-share-library`**项目中的**`/src/org/devops/`** 路径下 创建一个 **`SonarQube.groovy`** 共享工具类。

```groovy
package org.devops

// 扫描代码
def sonarScan(sonarQuberServer, projectName, projectService, projectDesc, homePage, projectPath) {

    def servers = ["test": "SonarQube-Test-Server", "prod":"SonarQube-Prod-Server"]

    withSonarQubeEnv("${servers[sonarQuberServer]}") {
        def sonarScannerHome = tool "SonarQube"

        def projectClassPath = projectPath - "src" + "target/classes"
        sonarDate = sh returnStdout: true, script: "date +%Y%m%d%H%M%S"
        sonarDate = sonarDate - "\n"

        sh """
            ${sonarScannerHome}/bin/sonar-scanner \
            -Dsonar.projectKey=${projectService} \
            -Dsonar.projectName=${projectName} \
            -Dsonar.projectVersion=${sonarDate} \
            -Dsonar.login=admin \
            -Dsonar.password=admin \
            -Dsonar.ws.timeout=30 \
            -Dsonar.projectDescription="${projectDesc}" \
            -Dsonar.links.homepage="${homePage}" \
            -Dsonar.sources=${projectPath} \
            -Dsonar.sourceEncoding=UTF-8 \
            -Dsonar.java.binaries="${projectClassPath}" 
        """
    }

}

// 获取代码扫描后的状态
def getCodeQaStat(projectName) {
    apiUrl = "project_branches/list?project=${projectName}"
    response = sonarApiReq("GET", apiUrl, '')
    response = readJSON text: """${response.content}"""
    result = response["branches"][0]["status"]["qualityGateStatus"]
    return result
}

// 请求 SonarQuber Api
def sonarApiReq(reqType, reqUrl, reqBody) {
    def sonarServer = "http://192.168.2.167/api/"
    result = httpRequest authentication: 'sonarqube-admin-username-password',
            httpMode: reqType,
            contentType: "APPLICATION_JSON",
            consoleLogResponseBody: true,
            ignoreSsleErrors: true,
            requestBody: reqBody,
            url: "${sonarServer}/${reqUrl}"
    return result
}

```

### 2.5  在 GitLab 中 Jenkins-share-library 项目 ci.Jenkinsfile 文件添加代码扫描阶段

```groovy
#!groovy

// ------------- jenkins 共享库 -------------
@Library('jenkins-share-library') _
// 构建工具
def build = new org.devops.Build()
// 其他通用工具
def tools = new org.devops.Tools()
// 引入gitlab 工具
def gitlab = new org.devops.Gitlab()
// 引入邮件发送工具
def email = new org.devops.Email()
// 引入代码扫描工具
def sonarQube = new org.devops.SonarQube()


// ------------- 环境变量 -------------
String buildType = "${env.buildType}"
String buildShell = "${env.buildShell}"
String gitlabUrl = "${env.gitlabUrl}"
String branchName = "${env.branchName}"
String nodeLabel  = "${env.nodeLabel}"
String projectPath = "${env.projectPath}"
String projectService = "${env.projectService}"
String projectDesc = "${env.projectDesc}"
String sonarServer = "${env.sonarServer}"

if ("${runOpts}" == "GitlabPush") {
	branchName = branch - "refs/heads/"

	currentBuild.description = "Trigger by ${userName} ${branch}"
	gitlab.changeCommitStatus(projectId, commitSha, "running")
}

pipeline {

    agent { node { label "${nodeLabel}" } }

    stages {
	    stage("Gitlab Checkout Code") {
			steps {
				script {
					println("${branchName}")
					tools.printColorMsg("获取代码", "green")
					checkout([
						$class: 'GitSCM',
						branches: [[name: '*${branchName}']],
						doGenerateSubmoduleConfigurations: false,
						extensions: [],
						submoduleCfg: [],
						userRemoteConfigs: [
							[credentialsId: 'gitlab-root-username-password', url: '${gitlabUrl}']
						]
					])
				}
			}
		}
        stage("Build") {
            steps {
                script {
				    tools.printColorMsg("执行打包", "green")
                    build.buildByTypeAndShell(buildType, buildShell)
                }
            }
        }

// ------------------------ 重点：代码扫描阶段部分 ---------------------------------------
		stage("Scanner Java Code QA") {
			steps {
				script {
					 tools.printColorMsg("代码扫描", "green")
					 projectName = projectName + "-" + projectService
					 sonarQube.sonarScan(sonarServer, projectName, projectName, projectDesc, projectHomePage, projectPath)
					 tools.printColorMsg("获取扫描结果", "green")
					 result = sonarQube.getCodeQaStat(projectName)

					 if (result.toString() == "ERROR") {
						 email.sendTo("代码质量质量阈错误! 请及时修复", userEmail)
						 tools.printColorMsg("代码质量质量阈错误! 请及时修复!", "red")
						 error "代码质量不及格! 请及时修复!"
					 } else {
						 println(result)
						 tools.printColorMsg("代码质量质量已通过，优秀！！！！！", "green")
					 }
				}
			}
		}
// ------------------------------------------------------------------------------------

    }

	post {
		always {
			script {
				println("always")
				email.sendTo("流水线构建成功", userEmail)
			}
		}
		success {
			script {
				println("success")
				gitlab.changeCommitStatus(projectId, commitSha, "success")
				email.sendTo("流水线构建成功", userEmail)
			}
		}
		failure {
			script {
				println("failure")
				gitlab.changeCommitStatus(projectId, commitSha, "failed")
				email.sendTo("流水线失败了", userEmail)
			}
		}
		aborted {
			script {
				println("aborted")
				gitlab.changeCommitStatus(projectId, commitSha, "canceled")
				email.sendTo("流水线已被取消", userEmail)

			}
		}
	}
}

```



### 2.6 在 Jenkins 中 nx-smartcity-micro-service 流水线项目添加必要参数

####  2.6.1 General 配置 

- 在的**`ci.jenkinsfile`**中，多了以下环境变量配置， 这些配置用于代码扫描阶段。

  ```groovy
  String projectPath = "${env.projectPath}"
  String projectService = "${env.projectService}"
  String projectDesc = "${env.projectDesc}"
  String sonarServer = "${env.sonarServer}"
  ```

- 那么在Jenkins 的 **`nx-smartcity-micro-service`** 流水线项目中点击配置，勾选“**`参数化构建过程`**”，然后点击“**`添加参数`**”，选择“**`选项参数`**”，添加以下参数:
  - **`projectPath`**：项目SRC目录，用于扫描该代码目录。
    - /var/jenkins_home/workspace/nx-smartctiy-micro-service/plan-service-impl/src 
    - /home/docker/mnt/jenkins_home/workspace/nx-smartctiy-micro-service/plan-service-impl/src
  - **`projectService`**：项目名称，用于作为扫描后项目名称。
    - plan-service-impl
  - **`projectDesc`**：项目描述，用于作为扫描后项目描述。
    - 南雄-智慧城市-微服务-1.0
  - **`sonarServer`**：SonarQube 服务器，用于匹配正式SonarQuber服务器，还是测试SonarQuber服务器。
    - test
    - prod

<img src="./../../../statics/images/jenkins/integration/jenkins_project_param_01.png" alt="image-20200716164337011" style="zoom:100%;" />

<img src="./../../../statics/images/jenkins/integration/jenkins_project_param_02.png" alt="image-20200716164337011" style="zoom:100%;" />



#### 2.6.2 构建触发器配置

- 在 **`Generic Webhook Trigger`**中的 `Post content parameters`添加以下参数：
  - Variable： **`projectName`**
  - Expression：**`$.project.name`**
    - JSONPath
  - Variable：**`projectHomePage`**
  - Expression：**`$.project.homepage`**
    - JSONPath

<img src="./../../../statics/images/jenkins/integration/jenkins_project_param_03.png" alt="image-20200716170213437" style="zoom:100%;" />



### 2.7 在 Gitlab 中 nx-smarctiy 项目执行提交代码。

```bash
$ git add *
$ git commit -m "update README.md"
$ git push origin master
```

## 3. 查看执行效果

<img src="./../../../statics/images/jenkins/integration/jenkins_integration_sonarquber_scanner_show_01.png" alt="image-20200716170213437" style="zoom:100%;" />

<img src="./../../../statics/images/jenkins/integration/jenkins_integration_sonarquber_scanner_show_02.png" alt="image-20200716170213437" style="zoom:100%;" />

