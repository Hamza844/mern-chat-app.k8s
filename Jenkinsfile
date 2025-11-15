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
                    def scannerHome = tool 'sonar-scnaeer'
                    
                    // Install Node.js if not present (inline)
                    sh '''
                        if ! command -v node &> /dev/null; then
                            echo "Node.js not found, installing..."
                            curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
                            sudo apt-get install -y nodejs
                        fi
                        echo "Node.js version: $(node --version)"
                    '''
                    
                    withSonarQubeEnv('SonarQube') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \\
                              -Dsonar.projectKey=${PROJECT_KEY} \\
                              -Dsonar.sources=. \\
                              -Dsonar.host.url=${SONAR_HOST_URL} \\
                              -Dsonar.exclusions=**/node_modules/**,**/dist/**
                        """
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
}