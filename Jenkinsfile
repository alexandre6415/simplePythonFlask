pipeline {
    agent any 

    environment {
        IMAGE-TAG= "0.${BUILD_ID}"
        DOCKER_IMAGE = "simple-python-flask"
    }
    stages {
        stage("Build") {
            steps {
                echo "Building the project..."
                sh "docker build -t simple-python-flask:${IMAGE-TAG} ."
            }
        }
        stage("Test"){

            steps { 
                sh "docker run -itd --name simple-python-flask-${IMAGE-TAG} --rm simple-python-flask:${IMAGE-TAG}"

                sh "docker exec -it simple-python-flask-${IMAGE-TAG} nosetests --with-xunit --with-coverage --cover-package=project test_users.py"
            }
        }
    }
}