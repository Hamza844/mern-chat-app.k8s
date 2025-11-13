pipeline {
    agent any

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

        stage('File scan') {
            steps {
                echo 'Running Trivy file system scan...'
                sh 'trivy fs .'
                echo 'File scan completed successfully!'
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
