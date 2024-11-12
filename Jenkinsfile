pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "aynuod/sonarpipelinedemo:latest"
        DOCKER_USERNAME = 'aynuod'
        DOCKER_PASSWORD = 'dckr_pat_GZx1tl4V1jCt0PSz_JZxrrXbL2s'
        APP_PORT = '8081'  // Port pour le déploiement
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
            // Remplacez "votre_mot_de_passe" par votre mot de passe Docker explicite
            bat "dckr_pat_GZx1tl4V1jCt0PSz_JZxrrXbL2s | docker login -u aynuod --password-stdin"
            bat "docker push ${DOCKER_IMAGE}"
            bat "docker logout"
            echo "Docker push completed"
        }
    }
}


        stage('Deploy Container') {
            steps {
                script {
                    echo "Starting deployment..."
                    // Arrêt et suppression du conteneur existant s'il existe
                    bat """
                        docker ps -f name=sonarpipelinedemo -q | findstr . && docker stop sonarpipelinedemo || echo "Container not running"
                        docker ps -a -f name=sonarpipelinedemo -q | findstr . && docker rm sonarpipelinedemo || echo "No container to remove"
                    """
                    
                    // Déploiement du nouveau conteneur
                    bat """
                        docker run -d ^
                        --name sonarpipelinedemo ^
                        -p ${APP_PORT}:8080 ^
                        ${DOCKER_IMAGE}
                    """
                    echo "Deployment completed. Application running on port ${APP_PORT}"
                }
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline terminé.'
            script {
                bat "docker system prune -f"
            }
        }
        success {
            echo "Application déployée avec succès! Accessible sur http://localhost:${APP_PORT}"
        }
        failure {
            echo 'Pipeline échoué.'
            // Nettoyage en cas d'échec
            script {
                bat """
                    docker ps -f name=sonarpipelinedemo -q | findstr . && docker stop sonarpipelinedemo || echo "No container to stop"
                    docker ps -a -f name=sonarpipelinedemo -q | findstr . && docker rm sonarpipelinedemo || echo "No container to remove"
                """
            }
        }
    }
}
