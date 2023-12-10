pipeline {
    agent any
    environment {
        JAVA_HOME = '/var/lib/jenkins/jdk-17'
        PATH = "$JAVA_HOME/bin:$PATH"
    }
    tools {
        maven 'maven'
    }
    stages {
        stage('Download and Install OpenJDK') {
            steps {
                script {
                    sh 'wget https://download.java.net/java/GA/jdk17/0d483333a00540d886896bac774ff48b/35/GPL/openjdk-17_linux-x64_bin.tar.gz'
                    sh 'tar -xvf openjdk-17_linux-x64_bin.tar.gz -C /var/lib/jenkins/'
                    sh 'chmod -R 755 /var/lib/jenkins/jdk-17'
                }
            }
        }
        stage('Build Maven') {
            steps {
                script {
                    withEnv(['JAVA_HOME=/var/lib/jenkins/jdk-17', "PATH=${tool 'maven'}/bin:$PATH"]) {
                        checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/rihabbelhadj/Devops.git']])
                        sh 'mvn clean install -U'
                    }
                }
            }
        }
        stage('Test') {
            steps {
                script {
                    sh './mvnw test'
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t rihabbelhadj/devops-integration .'
                }
            }
        }
        stage('Push Image to Hub') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'docker', variable: 'dockerhubpwd')]) {
                        sh "echo ${dockerhubpwd} | docker login -u rihabbelhadj --password-stdin"
                    }
                    sh 'docker push rihabbelhadj/devops-integration'
                }
            }
        }
    }
    post {
        success {
            echo 'Build and tests passed.'
        }
        failure {
            echo 'Build or tests failed.'
        }
    }
}
