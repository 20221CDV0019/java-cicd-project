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
                bat 'docker build -t 20221cdv0019/java-cicd-project:latest -t 20221cdv0019/java-cicd-project:%BUILD_NUMBER% .'
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

                    bat 'docker push 20221cdv0019/java-cicd-project:%BUILD_NUMBER%'

                    bat 'docker logout'
                }
            }
        }

        stage('Docker Deploy') {
            steps {
                bat 'docker stop java-app || exit 0'
                bat 'docker rm java-app || exit 0'

                bat 'docker run -d -p 8080:8080 --name java-app 20221cdv0019/java-cicd-project:latest'

                powershell '''
                    Write-Host "Waiting for application health check..."

                    $healthUrl = "http://localhost:8080/actuator/health"
                    $healthy = $false

                    for ($i = 1; $i -le 30; $i++) {
                        try {
                            $response = Invoke-WebRequest `
                                -Uri $healthUrl `
                                -UseBasicParsing `
                                -TimeoutSec 3

                            Write-Host "Health check response: $($response.Content)"

                            if ($response.StatusCode -eq 200 -and $response.Content -match '"status"\s*:\s*"UP"') {
                                $healthy = $true
                                Write-Host "Application is healthy!"
                                break
                            }
                        }
                        catch {
                            Write-Host "Application is not ready yet... attempt $i/30"
                        }

                        Start-Sleep -Seconds 2
                    }

                    if (-not $healthy) {
                        Write-Host "Application health check failed."
                        docker logs java-app
                        throw "Docker application did not become healthy."
                    }
                '''
            }
        }
    }
}