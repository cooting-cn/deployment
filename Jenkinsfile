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
  
    
    //声明流程
    stages {
        stage('从 gitlab 中拉取代码') {

      steps {
        script {

withCredentials([usernamePassword(credentialsId: "${GITID}", usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
branches = sh(script: 'git ls-remote -h -t https://${USERNAME}:${PASSWORD}@"${GITURL}" | awk "{print \\$2}"| sed "s#refs/heads/##;s#refs/tags/##"|tac', returnStdout: true).trim()
}
        echo "已有分支: ${branches} "
        env.BRANCH = input message: '请选择tag', ok: '确定', parameters: [choice(name: 'tag标签', choices: "${branches}", description: '默认5分钟,超时自动关闭')]
        echo "已经选着了分支: ${BRANCH} "
        
        
                }

                checkout([$class: 'GitSCM',
                          branches: [[name: "${BRANCH}"]],
                          doGenerateSubmoduleConfigurations: false,
                          extensions: [],
                          gitTool: 'Default',
                          submoduleCfg: [],
                          userRemoteConfigs: [[url: "${env.GITURL}",credentialsId: "${env.GITID}"]]
                        ])
        sh "cat main.go"
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
    

    
  }
}