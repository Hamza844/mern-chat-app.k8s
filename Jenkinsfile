pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('sonar')  // Changed from 'SONAR_TOKEN' to 'sonar'
        SONAR_HOST_URL = 'http://143.198.122.139:9000'
        PROJECT_KEY = 'mern-chat-app'
    }

    stages {
        stage('Install Tools') {
            steps {
                echo '🚀 Installing Docker and Trivy...'
                sh 'chmod +x install.sh && ./install.sh'
            }
        }

        stage('Verify Installation') {
            steps {
                echo '🔍 Verifying tools...'
                sh '''
                    echo "Docker: $(docker --version)"
                    echo "Trivy: $(trivy --version)"
                '''
            }
        }

        stage('Checkout') {
            steps {
                echo '📥 Checking out code...'
                checkout scm
            }
        }

        stage('Security Scan') {
            steps {
                echo '🔍 Running Trivy file system scan...'
                sh 'trivy fs --severity HIGH,CRITICAL .'
            }
        }

        stage('Code Quality') {
            steps {
                echo '📊 Running SonarQube analysis...'
                script {
                    def scannerHome = tool 'sonars-cnaeer'  // Use the tool name from Jenkins configuration
                    withSonarQubeEnv('SonarQube') {  // Changed to match your SonarQube server name
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                              -Dsonar.projectKey=${PROJECT_KEY} \
                              -Dsonar.sources=. \
                              -Dsonar.host.url=${SONAR_HOST_URL} \
                              -Dsonar.login=${SONAR_TOKEN} \
                              -Dsonar.exclusions=**/node_modules/**,**/dist/**
                        """
                    }
                }
            }
        }
    }
}