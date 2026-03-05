pipeline {
    agent any

    environment {
        // REMARQUE : Remplacez 'hmagroun' par votre propre nom d'utilisateur Docker Hub
        DOCKER_REPO = "hmagroun/composetest" 
        TEST_SERVER = "useradm@192.168.56.42"
    }

    stages {
        stage('Build Image') {
            steps {
                script {
                    def imageName = "${DOCKER_REPO}:${env.BUILD_NUMBER}"
                    echo "Building Docker image: ${imageName}"
                    sh "docker build -t ${imageName} ."
                    env.IMAGE_FULL_NAME = imageName
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', 
                passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USER')]) {
                    sh "docker login -u ${DOCKER_USER} -p ${DOCKER_PASSWORD}"
                    sh "docker push ${env.IMAGE_FULL_NAME}"
                    sh "docker tag ${env.IMAGE_FULL_NAME} ${DOCKER_REPO}:latest"
                    sh "docker push ${DOCKER_REPO}:latest"
                    sh "docker logout"
                }
            }
        }

        stage('Deploy on Test Server') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ssh-test-server', keyFileVariable: 'SSH_KEY')]) {
                    echo "Copying compose-prod.yaml to test server..."
                    sh "scp -i ${SSH_KEY} -o StrictHostKeyChecking=no compose-prod.yaml ${TEST_SERVER}:/home/useradm/"

                    echo "Executing deployment via SSH..."
                    sh "ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no ${TEST_SERVER} 'docker pull ${DOCKER_REPO}:latest'"
                    sh "ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no ${TEST_SERVER} 'docker compose -f /home/useradm/compose-prod.yaml up -d --force-recreate'"
                }
            }
        }
    }
}
