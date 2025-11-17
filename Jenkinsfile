pipeline {
    agent any

    environment {
        // SonarQube Configuration
        SONAR_TOKEN = credentials('Sonar')
        SONAR_HOST_URL = 'http://98.94.55.71:9000'
        PROJECT_KEY = 'mern-chat-app'

        // Docker Configuration
        DOCKERHUB_CREDS = credentials('dockerhub-creds')
        IMAGE_NAME = "mernchat-app"
        IMAGE_TAG = "${env.BUILD_NUMBER}" // Use build number for versioning
        
        // Kubernetes Configuration
        KUBECONFIG = '/var/lib/jenkins/.kube/config'
        K8S_NAMESPACE = 'prod'
        HELM_RELEASE = 'mern-chatapp-prod'
        HELM_CHART_PATH = '/home/helm/node-app'
    }

    stages {
        stage('Checkout') {
            steps {
                echo '📥 Checking out code...'
                checkout scm
            }
        }

        stage('Security Scan - Filesystem') {
            steps {
                echo '🔍 Running Trivy filesystem scan (HIGH,CRITICAL)...'
                sh 'trivy fs --severity HIGH,CRITICAL --exit-code 0 . || true'
            }
        }

        stage('SonarQube Code Quality') {
            steps {
                echo '📊 Running SonarQube analysis...'
                script {
                    def scannerHome = tool 'sonar-scanner'
                    
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
                    waitForQualityGate abortPipeline: false
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '🐳 Building Docker image...'
                script {
                    sh """
                        # Build image with build number tag
                        docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                        
                        # Tag with DockerHub username and build number
                        docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${DOCKERHUB_CREDS_USR}/${IMAGE_NAME}:${IMAGE_TAG}
                        
                        # Also tag as latest
                        docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${DOCKERHUB_CREDS_USR}/${IMAGE_NAME}:latest
                        
                        echo "✅ Docker image built successfully: ${DOCKERHUB_CREDS_USR}/${IMAGE_NAME}:${IMAGE_TAG}"
                    """
                }
            }
        }

        stage('Security Scan - Docker Image') {
            steps {
                echo '🔍 Scanning Docker image with Trivy...'
                script {
                    sh """
                        trivy image --format html --output trivy-report.html \
                          --severity HIGH,CRITICAL \
                          ${DOCKERHUB_CREDS_USR}/${IMAGE_NAME}:${IMAGE_TAG} || true
                    """
                }
            }
        }

        stage('Publish Security Report') {
            steps {
                echo '📄 Publishing Trivy HTML report to Jenkins...'
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

        stage('Push Docker Image to Registry') {
            steps {
                echo "📤 Pushing Docker image to Docker Hub..."
                script {
                    sh """
                        # Login to Docker Hub
                        echo "${DOCKERHUB_CREDS_PSW}" | docker login -u "${DOCKERHUB_CREDS_USR}" --password-stdin
                        
                        # Push with build number tag
                        docker push ${DOCKERHUB_CREDS_USR}/${IMAGE_NAME}:${IMAGE_TAG}
                        
                        # Push latest tag
                        docker push ${DOCKERHUB_CREDS_USR}/${IMAGE_NAME}:latest
                        
                        echo "✅ Image pushed: ${DOCKERHUB_CREDS_USR}/${IMAGE_NAME}:${IMAGE_TAG}"
                        echo "✅ Image pushed: ${DOCKERHUB_CREDS_USR}/${IMAGE_NAME}:latest"
                    """
                }
            }
        }

        stage('Deploy to Kubernetes with Helm') {
            steps {
                echo '🚀 Deploying to Kubernetes using Helm...'
                script {
                    sh """
                        # Set Kubernetes config explicitly
                        export KUBECONFIG=${KUBECONFIG}
                        export HOME=/var/lib/jenkins
                        
                        # Verify Kubernetes connectivity
                        echo "========================================="
                        echo "Kubernetes Cluster Information"
                        echo "========================================="
                        kubectl cluster-info
                        kubectl get nodes
                        echo ""
                        
                        # Navigate to Helm chart directory
                        cd ${HELM_CHART_PATH}
                        
                        # Validate Helm chart
                        echo "========================================="
                        echo "Validating Helm Chart"
                        echo "========================================="
                        helm lint .
                        echo ""
                        
                        # Show current chart info
                        helm show chart .
                        echo ""
                        
                        # Deploy application with Helm
                        echo "========================================="
                        echo "Deploying Application: ${HELM_RELEASE}"
                        echo "Namespace: ${K8S_NAMESPACE}"
                        echo "Image: ${DOCKERHUB_CREDS_USR}/${IMAGE_NAME}:${IMAGE_TAG}"
                        echo "========================================="
                        
                        helm upgrade --install ${HELM_RELEASE} . \
                          --namespace ${K8S_NAMESPACE} \
                          --create-namespace \
                          --set image.repository=${DOCKERHUB_CREDS_USR}/${IMAGE_NAME} \
                          --set image.tag=${IMAGE_TAG} \
                          --set image.pullPolicy=Always \
                          --wait \
                          --timeout 10m \
                          --atomic \
                          --cleanup-on-fail
                        
                        echo ""
                        echo "========================================="
                        echo "Waiting for Deployment Rollout"
                        echo "========================================="
                        kubectl rollout status deployment/${HELM_RELEASE} -n ${K8S_NAMESPACE} --timeout=300s
                        
                        echo ""
                        echo "========================================="
                        echo "✅ Deployment Status"
                        echo "========================================="
                        kubectl get all -n ${K8S_NAMESPACE}
                        echo ""
                        kubectl get pods -n ${K8S_NAMESPACE} -o wide
                        echo ""
                        
                        # Get deployed services
                        echo "========================================="
                        echo "📊 Service Information"
                        echo "========================================="
                        kubectl get svc -n ${K8S_NAMESPACE}
                        
                        echo ""
                        echo "✅ Deployment completed successfully!"
                        echo "📦 Deployed Version: ${IMAGE_TAG}"
                    """
                }
            }
        }
    }

    post {
        always {
            echo '🧹 Cleaning up local Docker images...'
            sh """
                docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true
                docker rmi ${DOCKERHUB_CREDS_USR}/${IMAGE_NAME}:${IMAGE_TAG} || true
                docker logout || true
            """
        }
        success {
            echo '✅ ========================================='
            echo '✅ Pipeline completed successfully!'
            echo '✅ ========================================='
            echo "✅ Image: ${DOCKERHUB_CREDS_USR}/${IMAGE_NAME}:${IMAGE_TAG}"
            echo "✅ Deployed to: ${K8S_NAMESPACE} namespace"
            echo '✅ ========================================='
        }
        failure {
            echo '❌ ========================================='
            echo '❌ Pipeline failed!'
            echo '❌ Please check the logs above for details'
            echo '❌ ========================================='
        }
    }
}