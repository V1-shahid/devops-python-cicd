pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/V1-shahid/devops-python-cicd.git'
            }
        }

        stage('Check Python') {
            steps {
                sh 'python3 --version'
                sh 'pip3 --version'
            }
        }
    }
}