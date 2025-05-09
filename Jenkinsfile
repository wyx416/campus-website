pipeline {
    agent any
    
    environment {
        GIT_REPO = "https://github.com/school-campus/website.git"  // 代码仓库
        DEPLOY_IP = "192.168.61.40"         // 目标服务器IP
        WEB_DIR = "/var/www/html"           // 网站目录
        SSH_CREDENTIALS = "server-ssh-key"  // SSH密钥凭据ID
    }

    stages {
        // 阶段1：获取最新代码
        stage('Checkout') {
            steps {
                git branch: 'main', 
                url: env.GIT_REPO,
                credentialsId: 'git-login'  // Git账户凭据
            }
        }

        // 阶段2：构建静态网站（假设使用Hugo）
        stage('Build') {
            steps {
                sh 'hugo --minify'  // 生成静态文件到public目录
            }
        }

        // 阶段3：部署到Web服务器
        stage('Deploy') {
            steps {
                sshagent([env.SSH_CREDENTIALS]) {
                    sh """
                        # 同步文件到服务器
                        rsync -avz --delete ./public/ ${DEPLOY_IP}:${WEB_DIR}/
                        
                        # 重启Nginx服务
                        ssh ${DEPLOY_IP} "sudo systemctl reload nginx"
                    """
                }
                echo "🎉 已成功部署到校园服务器"
            }
        }
    }

    post {
        always {
            cleanWs()  // 清理工作空间
        }
        success {
            sh 'echo "网站部署成功！访问地址：http://${DEPLOY_IP}"'
        }
        failure {
            sh 'echo "❌ 部署失败，请检查构建日志"'
        }
    }
}
