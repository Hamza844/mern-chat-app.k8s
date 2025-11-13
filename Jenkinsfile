pipeline {
    agent any

    environment {
        // SonarQube server configured in Jenkins (Manage Jenkins -> Configure System)
        SONARQUBE_SERVER = 'SonarQube'
        SONAR_TOKEN = credentials('SONAR_TOKEN') // ID of the token you added in Jenkins
    }

    stages {
        stage('Install Declarative Tool') {
            steps {
                echo 'Installing declarative tools...'
                sh '''
                    chmod +x install.sh
                    ./install.sh
                '''
            }
        }

        stage('File Scan with Trivy') {
            steps {
                echo 'Running Trivy file system scan...'
                sh 'trivy fs .'
                echo 'File scan completed successfully!'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo 'Running SonarQube analysis...'
                withSonarQubeEnv('SonarQube') { 
                    sh "sonar-scanner -Dsonar.projectKey=mern-chat-app -Dsonar.sources=. -Dsonar.host.url=http://100.24.23.144:8080 -Dsonar.login=$SONAR_TOKEN"
                }
                echo 'SonarQube scan completed successfully!'
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline finished successfully!'
        }
        failure {
            echo '❌ Pipeline failed!'
        }
    }
}
