pipeline {
    agent any
    environment {
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Njoyayvan22/Projet-SE-AVANC-E.git'
            }
        }
        stage('Build') {
            steps {
                sh 'dotnet restore'
                sh 'dotnet build --configuration Release --no-restore'
            }
        }
        stage('Test') {
            steps {
                sh 'dotnet test --no-build --verbosity normal'
            }
        }
        stage('Deploy') {
            steps {
                sh 'dotnet publish --configuration Release --output ./publish'
                // Ajoutez ici des étapes de déploiement (SCP, Docker, etc.)
            }
        }
    }
}
