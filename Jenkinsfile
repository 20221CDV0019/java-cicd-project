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
                bat 'docker build -t java-cicd-project .'
            }
        }

        stage('Docker Deploy') {
            steps {
                bat 'docker stop java-app || exit 0'
                bat 'docker rm java-app || exit 0'
                bat 'docker run -d -p 8080:8080 --name java-app java-cicd-project'
            }
        }
    }
}