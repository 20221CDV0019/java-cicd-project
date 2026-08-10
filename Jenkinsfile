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
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    bat 'docker login -u %DOCKER_USERNAME% -p %DOCKER_PASSWORD%'
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