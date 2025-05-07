pipeline {
    agent any

    stages {
        // 阶段1：拉取代码
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/wyx416/campus-website.git'
            }
        }

        // 阶段2：构建静态网站（示例为Node.js项目）
        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }

        // 阶段3：部署到Web服务器
        stage('Deploy') {
            steps {
                sshPublisher(
                    publishers: [
                        sshPublisherDesc(
                            configName: 'web-server',
                            transfers: [
                                sshTransfer(
                                    sourcePath: 'dist/**',  // 构建产物路径
                                    remoteDirectory: '/var/www/html',
                                    execCommand: 'sudo systemctl restart nginx'
                                )
                            ]
                        )
                    ]
                )
            }
        }
    }

    // 构建后操作
    post {
        success {
            slackSend channel: '#deploy', message: '校园网站部署成功！'
        }
        failure {
            slackSend channel: '#deploy', message: '校园网站部署失败！'
        }
    }
}
