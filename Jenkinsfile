pipeline {
    agent any
    environment {
        // 全局配置
        GIT_REPO = 'git@github.com:your-org/campus-website.git'
        GIT_CREDENTIALS = 'github-ssh-key'
        DEPLOY_DEV_SERVER = 'deploy-user@dev.campus.edu'
        DEPLOY_PROD_SERVER = 'deploy-user@prod.campus.edu'
        WEB_ROOT_DEV = '/var/www/dev-site'
        WEB_ROOT_PROD = '/var/www/prod-site'
        DOCKER_REGISTRY = 'registry.campus.edu:5000'
        SLACK_CHANNEL = '#deploy-notifications'
    }
    options {
        timeout(time: 30, unit: 'MINUTES')  // 超时设置
        disableConcurrentBuilds()           // 禁止并发构建
    }
    stages {
        // ---------------------- 代码获取阶段 ----------------------
        stage('Checkout Code') {
            steps {
                script {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: 'main']],
                        extensions: [],
                        userRemoteConfigs: [[
                            url: env.GIT_REPO,
                            credentialsId: env.GIT_CREDENTIALS
                        ]]
                    ])
                    // 记录当前 Commit 信息
                    sh 'git log -1 --pretty=format:"%h - %an, %ar : %s" > commit_info.txt'
                    archiveArtifacts 'commit_info.txt'
                }
            }
        }

        // ---------------------- 代码质量检查 ----------------------
        stage('Code Linting') {
            steps {
                sh 'npm install eslint --global'  // Node.js 示例
                sh 'eslint src/**/*.js'           // 静态代码检查
            }
        }

        // ---------------------- 构建与测试 ----------------------
        stage('Build & Test') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'npm install'
                        sh 'npm run test:unit'     // 单元测试
                        junit 'test-results/**/*.xml'  // 收集测试报告
                    }
                }
                stage('Integration Tests') {
                    steps {
                        sh 'npm run test:integration'  // 集成测试
                    }
                }
            }
            post {
                success {
                    script {
                        // 构建 Docker 镜像
                        docker.build("${env.DOCKER_REGISTRY}/campus-site:${env.BUILD_NUMBER}")
                    }
                }
                failure {
                    slackSend(
                        channel: env.SLACK_CHANNEL,
                        color: 'danger',
                        message: "❌ Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
                    )
                }
            }
        }

        // ---------------------- 部署到开发环境 ----------------------
        stage('Deploy to Dev') {
            when {
                branch 'main'  // 仅 main 分支触发
            }
            steps {
                sshPublisher(
                    publishers: [
                        sshPublisherDesc(
                            configName: 'dev-server',
                            transfers: [
                                sshTransfer(
                                    sourceFiles: 'dist/**',
                                    removePrefix: 'dist',
                                    remoteDirectory: env.WEB_ROOT_DEV,
                                    execCommand: """
                                        sudo systemctl restart nginx
                                        docker push ${env.DOCKER_REGISTRY}/campus-site:${env.BUILD_NUMBER}
                                    """
                                )
                            ]
                        )
                    ]
                )
            }
            post {
                success {
                    input(
                        message: '是否部署到生产环境?',
                        ok: 'Confirm',
                        parameters: [choice(choices: 'Yes\nNo', name: 'DEPLOY_PROD')]
                    )
                }
            }
        }

        // ---------------------- 人工审核与生产部署 ----------------------
        stage('Deploy to Production') {
            when {
                expression { 
                    return params.DEPLOY_PROD == 'Yes' 
                }
            }
            steps {
                script {
                    // 使用 Ansible 进行生产环境部署
                    ansiblePlaybook(
                        playbook: 'deploy-prod.yml',
                        inventory: 'inventory/prod.hosts',
                        extraVars: [
                            build_number: env.BUILD_NUMBER,
                            docker_registry: env.DOCKER_REGISTRY
                        ]
                    )
                }
            }
        }

        // ---------------------- 回滚机制 ----------------------
        stage('Rollback (Optional)') {
            when {
                expression { 
                    return currentBuild.result == 'FAILURE' 
                }
            }
            steps {
                script {
                    // 回滚到上一个稳定版本
                    sh '''
                        git revert HEAD
                        git push origin main
                    '''
                    slackSend(
                        channel: env.SLACK_CHANNEL,
                        color: 'warning',
                        message: "⚠️ 回滚触发: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
                    )
                }
            }
        }
    }

    // ---------------------- 全局后处理 ----------------------
    post {
        always {
            // 清理工作空间
            cleanWs()
            // 发送构建报告
            emailext (
                subject: "构建结果: ${currentBuild.result} - ${env.JOB_NAME}",
                body: """
                    Build URL: ${env.BUILD_URL}
                    Commit Info: ${readFile('commit_info.txt')}
                """,
                to: 'dev-team@campus.edu'
            )
        }
        success {
            slackSend(
                channel: env.SLACK_CHANNEL,
                color: 'good',
                message: "✅ 部署成功: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            )
        }
        failure {
            slackSend(
                channel: env.SLACK_CHANNEL,
                color: 'danger',
                message: "❌ 部署失败: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            )
        }
    }
}
