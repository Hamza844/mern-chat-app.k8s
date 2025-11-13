pipeline {
    agent {
        docker {
            image 'ubuntu:22.04'
            args '--privileged -v /var/run/docker.sock:/var/run/docker.sock'
        }
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
    }
}