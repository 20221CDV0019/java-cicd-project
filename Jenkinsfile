stage('Docker Push') {
    steps {
        withCredentials([
            usernamePassword(
                credentialsId: 'dockerhub-credentials',
                usernameVariable: 'DOCKER_USERNAME',
                passwordVariable: 'DOCKER_PASSWORD'
            )
        ]) {

            bat 'docker context use desktop-linux'

            bat 'echo %DOCKER_PASSWORD% | docker login -u %DOCKER_USERNAME% --password-stdin'

            timeout(time: 10, unit: 'MINUTES') {
                bat 'docker push %DOCKER_USERNAME%/java-cicd-project:latest'
            }

            bat 'docker logout'
        }
    }
}