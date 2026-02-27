pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "abhimukh/flask-app"
    }

    stages {

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_IMAGE}:${BUILD_NUMBER}")
                }
            }
        }

        stage('Push Image') {
            steps {
                script {
                    docker.withRegistry('', 'dockerhub-creds') {
                        dockerImage.push("${BUILD_NUMBER}")
                    }
                }
            }
        }
    }
}