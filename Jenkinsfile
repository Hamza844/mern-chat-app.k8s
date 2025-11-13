pipeline {
    agent any

    tools {
        'org.sonarsource.scanner.cli:sonar-scanner-cli' 'SonarScanner'
    }

    environment {
        SONAR_TOKEN = credentials('SONAR_TOKEN')
    }

    stages {
        stage('Install') {
            steps {
                sh 'chmod +x install.sh && ./install.sh'
            }
        }

        stage('Trivy Scan') {
            steps {
                sh 'trivy fs .'
            }
        }

        stage('SonarQube') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh 'sonar-scanner -Dsonar.projectKey=mern-chat-app -Dsonar.sources=.'
                }
            }
        }
    }
}