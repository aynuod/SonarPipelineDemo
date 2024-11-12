pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "aynuod/sonarpipelinedemo:latest"
        DOCKER_CREDENTIALS_ID = 'dockerhub_credentials'  // Configurez vos identifiants dans Jenkins
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/aynuod/SonarPipelineDemo.git', branch: 'main'
            }
        }

        stage('Compile') {
            steps {
                // Assurez-vous que Maven est installé et configuré dans Jenkins
                bat 'mvn clean compile'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'sonar-scanner'
                    withSonarQubeEnv('SonarQube') {
                        bat """
                            ${scannerHome}\\bin\\sonar-scanner.bat ^
                            -Dsonar.projectKey=stage ^
                            -Dsonar.host.url=http://localhost:9000 ^
                            -Dsonar.login=sqa_17ad9a4f6cd4022e49fdd58611ce3e18488859c8 ^
                            -Dsonar.sources=./src ^
                            -Dsonar.java.binaries=./target/classes
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}")
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', "${DOCKER_CREDENTIALS_ID}") {
                        docker.image("${DOCKER_IMAGE}").push()
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline terminé.'
        }
        failure {
            echo 'Pipeline échoué.'
        }
        success {
            echo 'Pipeline réussi.'
        }
    }
}
