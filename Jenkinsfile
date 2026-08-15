pipeline {

    agent any

    environment {
        DOCKER_IMAGE = "20221cdv0019/java-cicd-project"
        DOCKER_TAG = "${BUILD_NUMBER}"
        DOCKER_CREDENTIALS = "dockerhub-pat"
    }

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
                bat """
                    docker build -t %DOCKER_IMAGE%:latest .
                    docker tag %DOCKER_IMAGE%:latest %DOCKER_IMAGE%:%DOCKER_TAG%
                """
            }
        }

        stage('Docker Push') {
            steps {

                withCredentials([
                    string(
                        credentialsId: "${DOCKER_CREDENTIALS}",
                        variable: 'DOCKER_PAT'
                    )
                ]) {

                    powershell '''
                        Write-Host "Logging in to Docker Hub..."

                        $tempFile = Join-Path $env:TEMP "docker-pat-$env:BUILD_NUMBER.txt"

                        try {
                            [System.IO.File]::WriteAllText(
                                $tempFile,
                                $env:DOCKER_PAT,
                                [System.Text.UTF8Encoding]::new($false)
                            )

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

                    bat 'docker push %DOCKER_IMAGE%:latest'

                    bat 'docker push %DOCKER_IMAGE%:%DOCKER_TAG%'

                    bat 'docker logout'
                }
            }
        }

        stage('Trigger Render Deploy') {
            steps {

                withCredentials([
                    string(
                        credentialsId: 'render-deploy-hook',
                        variable: 'RENDER_DEPLOY_HOOK'
                    )
                ]) {

                    powershell '''
                        Write-Host "Triggering Render deployment..."

                        $response = Invoke-WebRequest `
                            -Uri $env:RENDER_DEPLOY_HOOK `
                            -Method Post `
                            -UseBasicParsing

                        Write-Host "Render response status: $($response.StatusCode)"

                        if ($response.StatusCode -lt 200 -or $response.StatusCode -ge 300) {
                            throw "Render deployment trigger failed."
                        }

                        Write-Host "Render deployment triggered successfully."
                    '''
                }
            }
        }

        stage('Docker Deploy') {
            steps {

                bat 'docker stop java-app || exit 0'

                bat 'docker rm java-app || exit 0'

                bat 'docker run -d -p 8080:8080 --name java-app %DOCKER_IMAGE%:latest'

                powershell '''
                    Write-Host "Waiting for application health check..."

                    $healthUrl = "http://localhost:8080/actuator/health"
                    $healthy = $false

                    for ($i = 1; $i -le 30; $i++) {

                        Write-Host "Health check attempt $i/30"

                        try {

                            $response = Invoke-WebRequest `
                                -Uri $healthUrl `
                                -UseBasicParsing `
                                -TimeoutSec 3

                            Write-Host "Health check response: $($response.Content)"

                            $health = $response.Content | ConvertFrom-Json

                            if ($response.StatusCode -eq 200 -and $health.status -eq "UP") {

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

                        Write-Host "Docker container status:"
                        docker ps -a --filter "name=java-app"

                        Write-Host "Docker container logs:"
                        docker logs java-app

                        throw "Docker application did not become healthy."
                    }
                '''
            }
        }
    }

    post {

        success {
            echo "=========================================="
            echo "CI/CD PIPELINE SUCCESSFUL"
            echo "=========================================="
            echo "Docker Image: ${DOCKER_IMAGE}:latest"
            echo "Build Number: ${BUILD_NUMBER}"
            echo "Application: http://localhost:8080"
            echo "Health: http://localhost:8080/actuator/health"
            echo "Render: https://java-cicd-project-unx5.onrender.com"
            echo "=========================================="
        }

        failure {
            echo "=========================================="
            echo "CI/CD PIPELINE FAILED"
            echo "=========================================="

            bat 'docker ps -a || exit 0'

            bat 'docker logs java-app || exit 0'

            echo "Please check the Jenkins console output."
            echo "=========================================="
        }

        always {
            echo "Pipeline completed."
        }
    }
}