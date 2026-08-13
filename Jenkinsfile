pipeline {
    agent any

    stages {

        stage('Test Docker as Jenkins') {
            steps {
                bat '''
                    whoami
                    docker version
                    docker context show
                '''
            }
        }
    }
}