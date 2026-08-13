pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/20221CDV0019/java-cicd-project.git'
            }
        }

        stage('Check Jenkins PAT') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'dockerhub-pat',
                        variable: 'DOCKER_PAT'
                    )
                ]) {

                    powershell '''
                        $sha = [System.Security.Cryptography.SHA256]::Create()

                        $hash = [BitConverter]::ToString(
                            $sha.ComputeHash(
                                [Text.Encoding]::UTF8.GetBytes($env:DOCKER_PAT)
                            )
                        ).Replace("-", "")

                        Write-Host "Jenkins PAT SHA256:"
                        Write-Host $hash
                    '''
                }
            }
        }
    }
}