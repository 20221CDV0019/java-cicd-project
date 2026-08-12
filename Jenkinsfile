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
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {

                    powershell '''
                        Write-Host "Docker username: $env:DOCKER_USERNAME"

                        if ([string]::IsNullOrEmpty($env:DOCKER_PASSWORD)) {
                            Write-Host "Docker password is EMPTY"
                            exit 1
                        }

                        Write-Host "Docker password is PRESENT"

                        $env:DOCKER_PASSWORD | docker login `
                            --username $env:DOCKER_USERNAME `
                            --password-stdin

                        if ($LASTEXITCODE -ne 0) {
                            exit $LASTEXITCODE
                        }
                    '''

                    bat 'docker push %DOCKER_USERNAME%/java-cicd-project:latest'
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