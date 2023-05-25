
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
    GITURL = 'https://gitlab.isigning.cn/ops/cicd-demo.git'
    //调用的git用户
    GITID = 'huqing'
    //定义的服务名称
    NAME= "${JOB_BASE_NAME}"
    //获取文件夹名字,定义命名空间
    NS= sh(script: 'echo $JOB_NAME | cut -d/ -f1', returnStdout: true).trim()
    //定义的仓库
    IMG= 'registry.isigning.cn'
    //最终镜像打包镜像名字
    DOCKERIMG = "${env.IMG}/cicd/${env.NAME}:${params.TAG}-$BUILD_NUMBER-${TIMESTAMP}"
    //引用k8s config
    KUBECONFIG = credentials('cicd-k8s')
    //用于存储当前时间戳，其值由一个Groovy表达式计算得出。  
    TIMESTAMP = sh(script: 'TZ=Asia/Shanghai date -d @`date +%s` "+%Y-%m-%d_%H-%M-%S"', returnStdout: true).trim()
  }

  //
  
  
  //声明选着标签变量TAG
    parameters {
        gitParameter name: 'TAG',
                     type: 'PT_BRANCH_TAG',
                     sortMode: 'DESCENDING_SMART',
                     selectedValue: 'TOP',
                     defaultValue: 'dev'
    }
  parameters {
    listGitBranches branchFilter: 'refs/heads/(.*)',
        defaultValue: 'dev',
        name: 'branch_name',
        type: 'PT_BRANCH',
        remoteURL: 'https://gitlab.isigning.cn/ops/cicd-demo.git',
        credentialsId: 'huqing',
        selectedValue: 'DEFAULT',
        sortMode: 'ASCENDING'
 }
    
    //声明流程
    stages {

        stage('从 gitlab 中拉取代码') {
            steps {
        git branch: "${params.branch_name}", credentialsId: 'huqing', url: 'https://gitlab.isigning.cn/ops/cicd-demo.git'
            }
        }
        
        
//声明docker打包
    stage('docker-build') {
      steps {
          sh """
        ls
        pwd
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
    
    
    
    
    //测试自动化压测
    stage('自动化,稳定性测试(暂未实现)') {
      steps {
      sh """

      echo "自动化测试,稳定性测试"
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
    
    stage('微信通知(暂未实现)') {
      steps {
      sh """
      echo "微信通知"
      """
      }
    }
    
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
