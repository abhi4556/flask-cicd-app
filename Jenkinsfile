pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "abhimukh/flask-app"
        MANIFESTS_REPO = "https://github.com/abhi4556/deployment_with_argo.git"
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
                    withCredentials([usernamePassword(credentialsId: 'github-creds', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PAT')]) {
                        sh """
                        git config --global user.email "abhijit.myworld@gmail.com"
                        git config --global user.name "abhi4556"

                        rm -rf ${MANIFESTS_DIR}
                        git clone https://${GIT_USER}:${GIT_PAT}@github.com/abhi4556/deployment_with_argo.git ${MANIFESTS_DIR}

                        cd ${MANIFESTS_DIR}
                        sed -i "s|image: abhimukh/flask-app:.*|image: abhimukh/flask-app:${BUILD_NUMBER}|" deployment.yaml
                        git add deployment.yaml
                        git commit -m "Update image tag to ${BUILD_NUMBER}" || echo "No changes to commit"
                        git push https://${GIT_USER}:${GIT_PAT}@github.com/abhi4556/deployment_with_argo.git
                        """
                    }
                }
            }
        }
}

}