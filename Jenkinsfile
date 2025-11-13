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
    }
}