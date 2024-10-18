pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                // Checkout code from Git
                git 'https://github.com/ManuCoppens/cicd-sample-app.git'

                // Build the Docker image
                script {
                    try {
                        sh 'docker build -t sampleapp:latest .'
                    } catch (Exception e) {
                        error "Docker build failed: ${e.message}"
                    }
                }
            }
        }

        stage('Run Application') {
            steps {
                // Stop and remove any existing container
                script {
                    sh 'docker stop samplerunning || true'
                    sh 'docker rm samplerunning || true'
                    
                    // Run the container
                    sh 'docker run -d -p 5050:5050 --name samplerunning sampleapp:latest'
                }
            }
        }

        stage('Acceptance Test') {
            steps {
                // Get the container IP address
                script {
                    def app_ip = sh(script: "docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' samplerunning", returnStdout: true).trim()
                    def jenkins_ip = sh(script: "hostname -I | awk '{print $1}'", returnStdout: true).trim()

                    // Acceptance test with curl
                    sh "curl http://${app_ip}:5050/ | grep 'You are calling me from ${jenkins_ip}'"
                }
            }
        }
    }

    post {
        always {
            // Clean up the container after the test
            sh 'docker stop samplerunning || true'
            sh 'docker rm samplerunning || true'
        }
    }
}