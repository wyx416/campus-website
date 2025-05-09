pipeline {
agent any

stages {
stage('Build') {
steps {
// 构建项目的命令
sh 'mvn clean package'
}
}
stage('Test') {
steps {
// 测试项目的命令
sh 'mvn test'
}
}
}
}
