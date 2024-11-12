pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "aynuod/sonarpipelinedemo:latest"
        DOCKER_CREDENTIALS_ID = 'dockerhub_credentials'
    }
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/aynuod/SonarPipelineDemo.git', branch: 'main'
            }
        }
        
        stage('Compile') {
            steps {
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
                    // Ajout de logging
                    echo "Starting Docker build..."
                    bat "docker build --no-cache -t ${DOCKER_IMAGE} ."
                    echo "Docker build completed"
                }
            }
        }
        
        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    echo "Starting Docker push..."
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        bat "echo %DOCKER_PASSWORD% | docker login -u %DOCKER_USERNAME% --password-stdin"
                        bat "docker push ${DOCKER_IMAGE}"
                        bat "docker logout"
                    }
                    echo "Docker push completed"
                }
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline terminé.'
            // Nettoyage
            script {
                bat "docker system prune -f"
            }
        }
        failure {
            echo 'Pipeline échoué.'
        }
        success {
            echo 'Pipeline réussi.'
        }
    }
}
