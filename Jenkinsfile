pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('Sonar')
        SONAR_HOST_URL = 'http://143.198.122.139:9000'
        PROJECT_KEY = 'mern-chat-app'
        DOCKERHUB_CREDS = credentials('dockerhub-creds')
        IMAGE_NAME = "hamza844/mernchat-app"   // Your Docker Hub repo
    }

    stages {

        stage('Checkout') {
            steps {
                echo '📥 Checking out code...'
                checkout scm
            }
        }

        stage('Security Scan (Filesystem)') {
            steps {
                echo '🔍 Running Trivy filesystem scan...'
                sh 'trivy fs --severity HIGH,CRITICAL . > fs-scan.txt || true'
            }
        }

        stage('SonarQube Code Quality') {
            steps {
                echo '📊 Running SonarQube analysis...'
                script {
                    def scannerHome = tool 'sonar-scanner'

                    withSonarQubeEnv('SonarQube') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                              -Dsonar.projectKey=${PROJECT_KEY} \
                              -Dsonar.sources=. \
                              -Dsonar.host.url=${SONAR_HOST_URL} \
                              -Dsonar.exclusions=**/node_modules/**,**/dist/**
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 3, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "🐳 Building Docker image..."
                sh """
                    docker build -t ${IMAGE_NAME}:latest .
                """
            }
        }

        stage('Trivy Scan Image') {
            steps {
                sh """
                    trivy image --severity HIGH,CRITICAL \
                    --format html -o trivy-image-report.html \
                    ${IMAGE_NAME}:latest || true
                """
            }
        }

        stage('Publish Trivy Report') {
            steps {
                echo '📄 Publishing HTML report...'
                publishHTML([
                    reportName: 'Trivy Scan Report',
                    reportDir: '.',
                    reportFiles: 'trivy-image-report.html',
                    keepAll: true
                ])
            }
        }

        stage('Push Docker Image') {
            steps {
                echo "📤 Pushing image to Docker Hub..."

                sh """
                    echo "${DOCKERHUB_CREDS_PSW}" | docker login -u "${DOCKERHUB_CREDS_USR}" --password-stdin
                    docker push ${IMAGE_NAME}:latest
                """
            }
        }
    }
}
