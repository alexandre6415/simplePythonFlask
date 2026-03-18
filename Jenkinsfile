pipeline {
    agent any

    environment {
        IMAGE_NAME = "alexandre6415/simplepythonflask"
        IMAGE_TAG  = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-credentials',
                    url: 'https://github.com/alexandre6415/simplePythonFlask.git'
            }
        }

        stage('Build Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Update Manifest') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'github-credentials',
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_TOKEN'
                )]) {
                    sh """
                        git config user.email "jenkins@ci.local"
                        git config user.name "Jenkins"
                        git fetch origin gitops
                        git checkout gitops
                        sed -i 's|image: alexandre6415/simplepythonflask:.*|image: alexandre6415/simplepythonflask:${IMAGE_TAG}|' manifest/deployment.yaml
                        git add manifest/deployment.yaml
                        git commit -m "ci: update image tag to ${IMAGE_TAG}"
                        git push https://${GIT_USER}:${GIT_TOKEN}@github.com/alexandre6415/simplePythonFlask.git gitops
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deploy disparado! ArgoCD vai sincronizar a tag ${IMAGE_TAG}"
        }
        failure {
            echo "❌ Pipeline falhou!"
        }
    }
}
