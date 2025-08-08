pipeline {
    agent any 

    environment {
        IMAGE_TAG= "0.${BUILD_ID}"
    }
    stages {
        stage("Build") {
            steps {
                echo "Building the project..."
                sh "docker build -t simple-python-flask:${IMAGE_TAG} ."
            }
        }
        stage("Test"){

            steps { 
                sh "docker run -itd --name simple-python-flask-${IMAGE_TAG} --rm simple-python-flask:${IMAGE_TAG}"

                sh "docker exec simple-python-flask-${IMAGE_TAG} nosetests --with-xunit --with-coverage --cover-package=project test_users.py"
            }
        }
    }

    post{
        success {
            echo "Build and tests were successful!"
            sh "docker stop simple-python-flask-${IMAGE_TAG}"
        }
        failure {
            echo "Build or tests failed."
            sh "docker stop simple-python-flask-${IMAGE_TAG}"
        }
        cleanup {
            echo "Cleaning up..."
            sh "docker rmi simple-python-flask:${IMAGE_TAG}"
        }
    }
}