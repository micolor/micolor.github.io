## 安装jenkins

```yml
version: '3'
services:
  jenkins:
    image: 'jenkins/jenkins:lts'
    container_name: jenkins
    restart: always
    environment:
      - TZ=Asia/Shanghai
    ports:
      - '8929:8080'
      # - '50000:50000'
    volumes:
      - '/etc/localtime:/etc/localtime'
      - './data:/var/jenkins_home' #jenkins_home
      - './maven:/usr/local/maven' #挂载maven
      - './node:/usr/local/node' #挂载node
```

## 测试

```shell
java -version
mvn -version
node -v
```

## pipeline 完整demo

```shell
pipeline {
    agent any
    options {
        //设置管道运行的超时时间，在此之后，Jenkins将中止管道
        timeout(time: 20, unit: 'MINUTES')
        // 失败重试次数
        retry(1)
        //输出时间戳
        timestamps()
    }
    environment {
        //构建分支 master、test
        // Brand='test'
        //定义Maven编译环境 dev/test/prod
        ACTIVE = "dev"
        //jar备份目录 
        JAR_BAK= "/var/jenkins_home/code/backup"
        PROJECT_DIR = "lucky-app"
        //jar包名称
        JAR_NAME = 'lucky-app.jar'
        //JOB所在根目录，JOB_NAME
        WORKSPACE_PATH = "/var/jenkins_home/code/edu/04-Development/0402-Source/trunk/edu-admin/"
    }
    stages {
      
		stage('Get Code'){
			steps{
			  dir("${WORKSPACE_PATH}") {
			     echo "拉取代码......"
				 sh 'git pull'
			   }
			}
		}
		
        stage('Build Package') {
             steps {
                  dir("${WORKSPACE_PATH}") {
                   echo "开始执行打包操作......."
                   //利用maven对项目进行build
                  sh "mvn -U -Dmaven.test.skip=true clean package"
                  sh "mv ${PROJECT_DIR}/target/*.jar  ${PROJECT_DIR}/target/${JAR_NAME}"
              }
             }
         }
         
        stage('Backup Jar'){
			steps{
			    dir("${WORKSPACE_PATH}") {
			      sh '''
			         if [ -f "${PROJECT_DIR}/target/${JAR_NAME}" ];then
			            cp -r ${PROJECT_DIR}/target/${JAR_NAME}  ${JAR_BAK}/$(date +%Y%m%d.%H:%M:%S)-${JAR_NAME}
			            echo "jar包备份成功..."
			         else
			            echo "备份的jar包不存在！"
			         fi
			      '''
				  sh '''
				  result=`find ${JAR_BAK}/ -mtime +10 -name "*-${JAR_NAME}"`
				  if [ -z "$result" ]
                  then
                      echo '不存在大于10天的 jar 包'
                  else
				      rm -rf $result
				  fi
				  '''
                } 
		    }
        }

        stage('Run Deploy') {
            steps {
              dir("${WORKSPACE_PATH}") {
                echo "开始执行Deploy操作......."
                sshPublisher(publishers: [sshPublisherDesc(configName: 'lucky', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/home', remoteDirectorySDF: false, removePrefix: 'lucky-app/target/', sourceFiles: 'lucky-app/target/*.jar')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
              }
            }
        }
    }
    post {
        success {
             echo "'${env.JOB_NAME} [${env.BUILD_NUMBER}]' 构建成功"
        }
        failure {
            echo "'${env.JOB_NAME} [${env.BUILD_NUMBER}]' 构建失败"
        }
    }
}
```

