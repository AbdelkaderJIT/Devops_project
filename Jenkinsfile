pipeline {
    agent any

    environment {
        IMAGE = 'gadraaii/devops-backend'
    }

    triggers {
        githubPush()          // trigger 2: webhook
        cron('H 2 * * *')     // trigger 3: every night around 2:00
    }

    stages {
        stage('Pull') {
            steps {
                git branch: 'main', url: 'https://github.com/AbdelkaderJIT/Devops_project.git'
            }
        }
        stage('Build') {
            steps {
                dir('backend') {
                    sh 'mvn clean compile'
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
        stage('Docker Image') {
            steps {
                dir('backend') {
                    sh 'docker build -t $IMAGE:$BUILD_NUMBER -t $IMAGE:latest .'
                }
            }
        }
        stage('Push Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                    sh 'docker push $IMAGE:$BUILD_NUMBER'
                    sh 'docker push $IMAGE:latest'
                }
            }
        }
        stage('Container') {
            steps {
                sh 'docker rm -f devops-backend || true'
                sh 'docker run -d --name devops-backend -p 8081:8080 $IMAGE:latest'
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }
    }
}
