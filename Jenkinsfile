pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "abhimukh/flask-app"
        MANIFESTS_REPO = "git@github.com:abhi4556/deployment_with_argo.git"
        MANIFESTS_DIR = "k8s-manifests"
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

        stage('Update Manifests') {
            steps {
                script {
                    sh """
                    # Configure git
                    git config --global user.email "abhijit.myworld@gmail.com"
                    git config --global user.name "abhi4556"

                    # Clone manifests repo
                    rm -rf ${MANIFESTS_DIR}
                    git clone ${MANIFESTS_REPO} ${MANIFESTS_DIR}

                    cd ${MANIFESTS_DIR}

                    # Update deployment image
                    sed -i "s|image: abhimukh/flask-app:.*|image: abhimukh/flask-app:${BUILD_NUMBER}|" deployment.yaml

                    # Commit & push
                    git add deployment.yaml
                    git commit -m "Update image tag to ${BUILD_NUMBER}"
                    git push origin main
                    """
                }
            }
        }
    }
}