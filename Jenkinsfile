pipeline {

    agent any
    stages {
        stage("Build")
        {
            steps {
                echo "Building the project..."
                // Add your build commands here
            }
        }steps{
            sh "docker build -t simple-python-flask"
        }

    }
}