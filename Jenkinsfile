pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('Sonar')
        SONAR_HOST_URL = 'http://143.198.122.139:9000'
        PROJECT_KEY = 'mern-chat-app'

        DOCKER_USERNAME = credentials('Docker_USERNAME')
        DOCKER_TOKEN = credentials('Docker_Token')
        IMAGE_NAME = "mernchat-app"
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
                echo '🔍 Running Trivy filesystem scan (HIGH,CRITICAL)...'
                // trivy already installed on server as you said
                sh 'trivy fs --severity HIGH,CRITICAL . || true'
            }
        }

        stage('SonarQube Code Quality') {
            steps {
                echo '📊 Running SonarQube analysis...'
                script {
                    def scannerHome = tool 'sonar-scanner'

                    // Ensure Node available for scanner if your project needs it
                    sh '''
                        if ! command -v node &> /dev/null; then
                            echo "Node not found, installing..."
                            curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
                            sudo apt-get install -y nodejs
                        fi
                        echo "Node version: $(node --version || true)"
                    '''

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
                echo '⏳ Waiting for SonarQube Quality Gate...'
                timeout(time: 5, unit: 'MINUTES') {
                    // set abortPipeline:true if you want to stop on failure
                    waitForQualityGate abortPipeline: false
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '🐳 Building Docker image...'
                script {
                    sh """
                        # build image
                        docker build -t ${IMAGE_NAME}:latest .
                        # tag image with your Docker Hub username
                        docker tag ${IMAGE_NAME}:latest ${DOCKER_USERNAME}/${IMAGE_NAME}:latest

                        # login and push (optional). If you don't want to push, comment the login/push lines.
                        echo "${DOCKER_TOKEN}" | docker login -u "${DOCKER_USERNAME}" --password-stdin
                        docker push ${DOCKER_USERNAME}/${IMAGE_NAME}:latest
                    """
                }
            }
        }

        stage('Trivy Scan Image') {
            steps {
                echo '🔍 Scanning Docker image with Trivy (image scan)...'
                script {
                    // Saves an HTML report file
                    sh """
                        trivy image --format html --output trivy-report.html ${DOCKER_USERNAME}/${IMAGE_NAME}:latest || true
                    """
                }
            }
        }

        stage('Publish Trivy Report') {
            steps {
                echo '📄 Publishing Trivy HTML report to Jenkins...'
                // requires the HTML Publisher Plugin to be installed in Jenkins
                publishHTML(target: [
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: '.',
                    reportFiles: 'trivy-report.html',
                    reportName: 'Trivy Security Scan Report'
                ])
            }
        }
        stage('Push Docker Image') {
            steps {
                echo "📤 Pushing image to Docker Hub..."

                sh """
                    echo "${DOCKER_TOKEN_PSW}" | docker login -u "${DOCKER_USERNAME_USR}" --password-stdin
                    docker push ${IMAGE_NAME}:latest
                """
            }
        }
    }
}
