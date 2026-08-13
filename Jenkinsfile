pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/20221CDV0019/java-cicd-project.git'
            }
        }

        stage('Build & Test') {
            steps {
                bat 'mvnw.cmd clean test'
            }
        }

        stage('Docker Build') {
            steps {
                bat 'docker build -t 20221cdv0019/java-cicd-project:latest .'
            }
        }

        stage('Docker Push') {
            steps {

                powershell '''
                    Write-Host "=== Jenkins Docker Environment ==="
                    whoami
                    docker version
                    docker context show
                    docker info
                    Write-Host "=================================="
                '''

                withCredentials([
                    string(
                        credentialsId: 'dockerhub-pat',
                        variable: 'DOCKER_PAT'
                    )
                ]) {

                    powershell '''
                        $env:DOCKER_PAT | docker login `
                            --username "20221cdv0019" `
                            --password-stdin

                        if ($LASTEXITCODE -ne 0) {
                            exit $LASTEXITCODE
                        }
                    '''

                    bat 'docker push 20221cdv0019/java-cicd-project:latest'

                    powershell '''
                        docker logout
                    '''
                }
            }
        }

        stage('Docker Deploy') {
            steps {
                bat 'docker stop java-app || exit 0'
                bat 'docker rm java-app || exit 0'

                bat 'docker run -d -p 8080:8080 --name java-app 20221cdv0019/java-cicd-project:latest'
            }
        }
    }
}