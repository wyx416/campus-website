pipeline {
  agent any
  stages {
    // 拉取代码
    stage('Checkout') {
      steps {
        git url: 'https://github.com/yourname/simple-web-app.git', branch: 'main'
      }
    }

    // 构建Docker镜像
    stage('Build') {
      steps {
        script {
          docker.build("my-web-app:${env.BUILD_ID}")
        }
      }
    }

    // 运行单元测试
    stage('Test') {
      steps {
        sh 'docker run my-web-app:${BUILD_ID} npm test'
      }
    }

    // 推送镜像到仓库
    stage('Push') {
      steps {
        script {
          docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-cred') {
            docker.image("my-web-app:${env.BUILD_ID}").push()
          }
        }
      }
    }

    // 部署到服务器
    stage('Deploy') {
      steps {
        sh '''
          docker stop live-app || true
          docker rm live-app || true
          docker run -d -p 80:80 --name live-app my-web-app:${BUILD_ID}
        '''
      }
    }
  }
}
