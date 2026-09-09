pipeline {
    agent any

    stages {

        stage('Check Python') {
            steps {
                sh '''
                    python3 --version
                    pip3 --version
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    python3 -m venv .venv
                    .venv/bin/pip install --upgrade pip
                    .venv/bin/pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                    .venv/bin/pytest
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t devops-python-cicd:${BUILD_NUMBER} .
                '''
            }
        }
        stage('Deploy') {
            steps {
                sh '''
                    docker rm -f 
                    devops-python-cicd-container || true
                    docker run -d --name 
                    devops-python-cicd-container -p 
                    5000:5000 devops-python-cicd:$
                    {BUILD_NUMBER}
                    '''
            }
        }
        stage('Health Check') {
            steps {
                sh '''
                    ddocker exec devops-python-cicd-container python -c "import urllib.request; r=urllib.request.urlopen('http://localhost:5000/health'); print(r.read().decode()); exit(0 if r.status == 200 else 1)"
                    '''
            }
        }
    }
}