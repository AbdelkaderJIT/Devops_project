pipeline {
    agent any

    triggers {
        githubPush()
    }

    stages {
        stage('Pull') {
            steps {
                git branch: 'main', url: 'https://github.com/AbdelkaderJIT/Devops_project.git'
            }
        }
        stage('Clean') {
            steps {
                dir('backend') {
                    sh 'mvn clean'
                }
            }
        }
        stage('Package') {
            steps {
                dir('backend') {
                    sh 'mvn package -DskipTests=true'
                }
            }
        }
    }
}
