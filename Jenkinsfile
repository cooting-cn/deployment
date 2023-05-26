
pipeline {
  agent {
    kubernetes {
      cloud 'kubernetes'
      slaveConnectTimeout 1200
      workspaceVolume emptyDirWorkspaceVolume()
      yaml """
kind: Pod
spec:
  containers:
    - name: jnlp
      image: 'registry.isigning.cn/aos_aliyun/inbound-agent:alpine-jdk11-npm-go'
      pullSecret: aoscript-registry
      tty: true 
      securityContext:
        runAsUser: 0
      volumeMounts:
        - mountPath: "/var/run/docker.sock"
          name: "docker-sock"
        - mountPath: "/usr/local/share/.cache"
          name: "yarn-che"
        - mountPath: "/root/go"
          name: "go-che"
  nodeSelector:
    cicd: ''
  imagePullSecrets:
    - name: aoscript-registry
  tolerations:
    - key: cicd
      operator: Equal
      effect: NoSchedule
  restartPolicy: Never
  volumes:
    - name:  "docker-sock"
      hostPath:
        path:  "/var/run/docker.sock"
    - name:  "yarn-che"
      hostPath:
        path:  "/opt/yarn/.cache"
    - name:  "go-che"
      hostPath:
        path:  "/opt/go"
      """
    }
  }
  //
  //全局变量，会在所有stage中生效
  //
  //

  environment {   
    //存储Git仓库的地址。
    GITURL = 'gitlab.isigning.cn/ops/cicd-demo.git'
    //调用的git用户
    GITID = 'huqing'
    //引用k8s config
    KUBECONFIG = credentials('cicd-k8s')
    //定义的仓库
    IMG= 'registry.isigning.cn/cicd'
    //定义的服务名称
    NAME= "${JOB_BASE_NAME}"
    //获取文件夹名字,定义命名空间
    NS= sh(script: 'echo $JOB_NAME | cut -d/ -f1', returnStdout: true).trim()
    //最终镜像打包镜像名字
//    DOCKERIMG = ''
    //用于存储当前时间戳，其值由一个Groovy表达式计算得出。  
    TIMESTAMP = sh(script: 'TZ=Asia/Shanghai date -d @`date +%s` "+%Y-%m-%d_%H-%M-%S"', returnStdout: true).trim()
  }


//声明流程
stages {
    
    stage('从 gitlab 中拉取代码') {
        //一分钟超时
      options {
        timeout(time: 1, unit: 'MINUTES') // 在此处添加超时选项
      }
      
      steps {
        script {

          //获取当前jenkins的git凭证,然后获取git仓库的分支tag,赋值成变量
          withCredentials([usernamePassword(credentialsId: "${GITID}", usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
          branches = sh(script: 'git ls-remote -h -t https://${USERNAME}:${PASSWORD}@"${GITURL}" | awk "{print \\$2}"| sed "s#refs/heads/##;s#refs/tags/##"|tac', returnStdout: true).trim()
          }
          
          echo "已有分支: ${branches} "
          //提供动态选着窗口,进行分支选着
          env.BRANCH = input message: '请选择tag', ok: '确定', parameters: [choice(name: 'tag标签', choices: "${branches}", description: '默认5分钟,超时自动关闭')]
          echo "已经选着了分支: ${BRANCH} "
          
          //最终镜像打包镜像名字,定义的全局变量
          env.DOCKERIMG="${env.IMG}/${env.NAME}:${BRANCH}-$BUILD_NUMBER-${TIMESTAMP}"

          echo "已经定义全局镜像:${env.DOCKERIMG}"

        }
        

        //拉取git仓库tag代码
        checkout([$class: 'GitSCM',
                  branches: [[name: "${BRANCH}"]],
                  doGenerateSubmoduleConfigurations: false,
                  extensions: [],
                  gitTool: 'Default',
                  submoduleCfg: [],
                  userRemoteConfigs: [[url: 'https://'+"${env.GITURL}",credentialsId: "${env.GITID}"]]
                ])
                
      }
    }
        
        
//声明docker打包
    stage('docker-build') {
      steps {
          sh """
        ls
        pwd
        go env -w GOPROXY=https://goproxy.cn,direct
        go build -o app
cat >dockerfile<<EOF
FROM registry.isigning.cn/cicd/golang:alpine-v1
COPY app /opt/
EOF
        cat dockerfile
        docker build -t  ${env.DOCKERIMG} .
        echo "Axzx@2022!" | docker login --username=aoxiongonline ${env.IMG} --password-stdin
        docker push ${env.DOCKERIMG}
         """
      }
    }
    
    //代码漏洞,img镜像扫描
    stage('漏洞,镜像扫描(暂未实现)') {
      steps {
      sh """
      echo "代码漏洞,img镜像扫描"
      """
      }
    }
    
    
    
    //部署到k8s
    stage('使用模板,部署或更新deply') {
      steps {
      git 'https://gitee.com/cooting/deployment.git'
      sh """
      #写入当前流水线变量
      export k8sNamespace=${env.NS}
      export Name=${env.NAME}
      export imageName=${env.DOCKERIMG}
      #把变量输出到模版文件
      envsubst < deployment/template.yaml|cat -
      envsubst < deployment/template.yaml|kubectl apply -f -
      kubectl get pod -n ${env.NS}  --kubeconfig $KUBECONFIG

      #kubectl set image deploy ${env.NAME} -n ${env.NS} ${env.NAME}=${env.DOCKERIMG}
      """
      }
    }
    
    //测试自动化压测
    stage('自动化,稳定性测试(暂未实现)') {
      steps {
      sh """
      echo "自动化测试,稳定性测试"
      """
      }
    }
    
    
    //灰度待定
    stage('灰度发布(暂未实现)') {
      steps {
      sh """
      echo "灰度"
      """
      }
    }
    //待定选着灰度阶段
//    stage('灰度是否到生产,默认超时时间20分钟') {
//        options {
//    timeout(time: 1, unit: 'MINUTES') // 在此处添加超时选项
//   }
//      input {
//        message "还继续么?"
//        ok "继续"
//        submitter ""
//        parameters {
//          string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
//        }
//    }
//    
//    
//      steps {
//        echo "Hello, ${PERSON}"
//      }
//    }
//    

    stage('微信通知') {
      steps {
        wxwork(
                robot: '1',
                type: 'markdown',
                text: [
                        "# 构建项目 ：${env.JOB_NAME}",
                        "- 地址 ：${env.JOB_URL}",
                        "- 流水线编号 ：[${currentBuild.displayName}](${env.BUILD_URL})点击跳转",
                        "- 持续时间：${currentBuild.durationString}".split("and counting")[0],
                        "- 执行人：${currentBuild.buildCauses.shortDescription}",
                        '> end'
                ]
        )
      sh """
      echo "微信通知"
      """
      }
    }
//成功失败后的反馈    
  }
  post {
    success{
        script{
            println("成功！！ ✅✅✅✅✅✅✅✅✅")
            currentBuild.description = "\n 构建成功"
        }
    }    
    failure {
        script{
            println("失败！！ ❌❌❌❌❌❌❌❌❌")
            currentBuild.description = "\n 构建失败"
        }

     }

    aborted {
        script{
            println("失败！！ ❌❌❌❌❌❌❌❌❌")
            currentBuild.description = "\n 构建失败"
        }
    }
    }
}
