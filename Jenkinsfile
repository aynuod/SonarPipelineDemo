pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/aynuod/JPA-Jenkins.git', branch: 'main'
            }
        }

        stage('Build') {
            steps {
                sh './mvnw clean install'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh './mvnw sonar:sonar \
                    -Dsonar.projectKey=spring-test \
                    -Dsonar.host.url=http://localhost:9000 \
                    -Dsonar.login=sqp_235be6134cd1bd216e123a0fb0e38f0b087c9dc2'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}
