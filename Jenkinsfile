pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('Sonar')
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
                    echo "Node.js: $(node --version || echo 'Not installed')"
                    echo "NPM: $(npm --version || echo 'Not installed')"
                '''
            }
        }
        
        stage('Install Node.js') {
            when {
                expression {
                    def result = sh(script: 'which node', returnStatus: true)
                    return result != 0
                }
            }
            steps {
                echo '📦 Installing Node.js...'
                sh '''
                    curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
                    sudo apt-get install -y nodejs
                    echo "Node.js installed: $(node --version)"
                    echo "NPM installed: $(npm --version)"
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
                sh 'trivy fs --severity HIGH,CRITICAL . || true'
            }
        }

        stage('Code Quality') {
            steps {
                echo '📊 Running SonarQube analysis...'
                script {
                    def scannerHome = tool 'sonars-cnaeer'
                    
                    withSonarQubeEnv('SonarQube') {
                        // Using single quotes to avoid Groovy interpolation
                        // Using -Dsonar.token instead of deprecated -Dsonar.login
                        sh '''
                            ${scannerHome}/bin/sonar-scanner \
                              -Dsonar.projectKey=${PROJECT_KEY} \
                              -Dsonar.sources=. \
                              -Dsonar.host.url=${SONAR_HOST_URL} \
                              -Dsonar.token=${SONAR_TOKEN} \
                              -Dsonar.exclusions=**/node_modules/**,**/dist/**
                        '''
                    }
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }
    }
    
    post {
        always {
            echo '🏁 Pipeline finished!'
        }
        success {
            echo '✅ Pipeline succeeded!'
        }
        failure {
            echo '❌ Pipeline failed! Check the logs above.'
        }
    }
}