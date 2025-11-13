pipeline {
    agent any

    stages {
        stage('Install Declarative Tool') {  // Fixed typo: "decalerative" → "Declarative"
            steps {
                echo 'Installing declarative tools...'
                sh '''
                    chmod +x install.sh
                    ./install.sh
                '''
            }
        }
    }
}