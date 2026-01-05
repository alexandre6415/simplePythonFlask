podTemplate(
    containers: [
        containerTemplate(name: 'maven', image: 'maven:3.8.1-jdk-8', command: 'sleep', args: '99d'),
        containerTemplate(
            name: 'docker',
            image: 'docker:latest',
            command: 'sleep',
            args: '99d',
            ttyEnabled: true
        ),
        containerTemplate(name: 'openjdk', image: 'eclipse-temurin:17-jdk', command: 'sleep', args: '99d')
    ],
    volumes: [ 
        hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock')
    ],
    serviceAccount: 'default'
){

    node(POD_LABEL) {
        container('docker') {

            stage("Clona Git") {
                git 'http://192.168.88.20:3000/alexandre/simplePythonFlask.git'
            }

            stage("Build") {
                sh """
                    echo "=== DEBUG ==="
                    whoami
                    id
                    ls -la /var/run/ || echo "No /var/run/"
                    ls -la /var/run/docker.sock || echo "No docker.sock"
                    echo "=== END DEBUG ==="
                    docker build -t simple-python-flask:${BUILD_ID} .
                """
            }

            stage("Test") {
                sh "docker run -itd --name simple-python-flask-${BUILD_ID} simple-python-flask:${BUILD_ID}"
                sh "docker exec simple-python-flask-${BUILD_ID} nosetests --with-xunit --with-coverage --cover-package=project test_users.py"
                sh "docker stop simple-python-flask-${BUILD_ID}"
                sh "docker rm simple-python-flask-${BUILD_ID}"
                sh "docker tag simple-python-flask:${BUILD_ID} 192.168.88.20:8082/simple-python-flask:${BUILD_ID}"
            }
        } 
        container('openjdk'){

            stage('SonarQube Analysis'){
                script {
                    def sonarScannerPath = tool 'SonarScanner'

                    withSonarQubeEnv ('SonarQube'){
                        sh """${sonarScannerPath}/bin/sonar-scanner \
                            -Dsonar.projectKey=courseCatalog \
                            -Dsonar.sources=. \
                            -Dsonar.javascript.enabled=false \
                            -Dsonar.typescript.enabled=false \
                            -Dsonar.css.enabled=false"""
                    }
                }
            }
        }

        container('docker') {
            stage("Push image") {
                script {
                    docker.withRegistry('http://192.168.88.20:8082', 'jenkins_docker') {
                        sh "docker push 192.168.88.20:8082/simple-python-flask:${BUILD_ID}"
                    }
                }
            }
        }
    }
}
