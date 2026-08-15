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

            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
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
                    string(
                        credentialsId: 'dockerhub-pat',
                        variable: 'DOCKER_PAT'
                    )
                ]) {

                    powershell '''
                        $tempFile = Join-Path $env:TEMP "docker-pat-$env:BUILD_NUMBER.txt"

                        try {
                            [System.IO.File]::WriteAllText(
                                $tempFile,
                                $env:DOCKER_PAT,
                                [System.Text.UTF8Encoding]::new($false)
                            )

                            Write-Host "Logging in to Docker Hub..."

                            cmd /c "docker login -u 20221cdv0019 --password-stdin < `"$tempFile`""

                            if ($LASTEXITCODE -ne 0) {
                                throw "Docker Hub login failed"
                            }

                            Write-Host "Docker Hub login successful"

                        }
                        finally {
                            if (Test-Path $tempFile) {
                                Remove-Item $tempFile -Force
                            }
                        }
                    '''

                    bat 'docker push 20221cdv0019/java-cicd-project:latest'

                    bat 'docker logout'
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