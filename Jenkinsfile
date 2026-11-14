pipeline {
    agent any

    environment {
        BACK_IMAGE  = 'gadraaii/devops-backend'
        FRONT_IMAGE = 'gadraaii/devops-frontend'
    }

    triggers {
        githubPush()
        cron('H 2 * * *')
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
        stage('Docker Images') {
            steps {
                dir('backend') {
                    sh 'docker build -t $BACK_IMAGE:$BUILD_NUMBER -t $BACK_IMAGE:latest .'
                }
                dir('frontend') {
                    sh 'docker build -t $FRONT_IMAGE:$BUILD_NUMBER -t $FRONT_IMAGE:latest .'
                }
            }
        }
        stage('Push Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                    sh 'docker push $BACK_IMAGE:$BUILD_NUMBER'
                    sh 'docker push $BACK_IMAGE:latest'
                    sh 'docker push $FRONT_IMAGE:$BUILD_NUMBER'
                    sh 'docker push $FRONT_IMAGE:latest'
                }
            }
        }
        stage('Containers') {
            steps {
                sh 'docker network create devops-net || true'
                sh 'docker start mysql-db || docker run -d --name mysql-db --network devops-net -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=gestionprojets -v mysql-data:/var/lib/mysql mysql:8'
                sh 'docker rm -f devops-backend devops-frontend || true'
                sh '''
                    docker run -d --name devops-backend --network devops-net \
                      --restart on-failure:10 -p 8081:8080 \
                      -e SPRING_DATASOURCE_URL="jdbc:mysql://mysql-db:3306/gestionprojets?createDatabaseIfNotExist=true" \
                      -e SPRING_DATASOURCE_USERNAME=root \
                      -e SPRING_DATASOURCE_PASSWORD=root \
                      -e SPRING_JPA_HIBERNATE_DDL_AUTO=update \
                      $BACK_IMAGE:latest
                '''
                sh 'docker run -d --name devops-frontend --network devops-net -p 4200:80 $FRONT_IMAGE:latest'
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }
    }
}
