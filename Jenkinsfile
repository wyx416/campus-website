pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package -Dmaven.test.skip=true'  # Maven构建
            }
        }
        stage('Docker Build') {
            steps {
                sh '''
                docker build -t campus-website:latest .
                docker stop campus-website || true
                docker rm campus-website || true
                docker run -d -p 80:8080 --name campus-website campus-website:latest
                '''
            }
        }
    }
}
