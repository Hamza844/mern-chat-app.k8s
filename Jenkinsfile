pipeline {
    agent any

    tools {
        'org.sonarsource.scanner.cli:sonar-scanner-cli' 'SonarScanner'
    }

    environment {
        SONAR_TOKEN = credentials('SONAR_TOKEN')
        SONAR_HOST_URL = 'http://143.198.122.139:9000'
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
                script {
                    try {
                        withSonarQubeEnv('sonarqube') {
                            sh """
                                sonar-scanner \
                                  -Dsonar.projectKey=mern-chat-app \
                                  -Dsonar.sources=. \
                                  -Dsonar.host.url=${SONAR_HOST_URL} \
                                  -Dsonar.login=${SONAR_TOKEN}
                            """
                        }
                    } catch (Exception e) {
                        echo "⚠️ SonarQube scan failed: ${e.message}"
                        echo "Continuing pipeline anyway..."
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }
    }
}