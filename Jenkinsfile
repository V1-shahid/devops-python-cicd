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

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    sh '''
                        docker login -u "$DOCKER_USER" -p "$DOCKER_PASSWORD"
                        docker tag devops-python-cicd:${BUILD_NUMBER} v1shahid/devops-python-cicd:${BUILD_NUMBER}
                        docker push v1shahid/devops-python-cicd:${BUILD_NUMBER}
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    # Save the currently running image as the previous version
                    CURRENT_IMAGE=$(docker inspect -f '{{.Config.Image}}' devops-python-cicd-container)

                    echo "Current running image: $CURRENT_IMAGE"
                    docker tag "$CURRENT_IMAGE" v1shahid/devops-python-cicd:prebious

                    #pull the new image
                    docker pull v1shahid/devops-python-cicd:${BUILD_NUMBER}

                    #Remove old container
                    docker rm -f devops-python-cicd-container || true


                    # Start new version
                    docker run -d \
                        --name devops-python-cicd-container \
                        -p 5000:5000 \
                        v1shahid/devops-python-cicd:${BUILD_NUMBER}
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    docker exec devops-python-cicd-container \
                        python -c "import urllib.request; r=urllib.request.urlopen('http://host.docker.internal:5000/health'); print(r.read().decode()); exit(0 if r.status == 200 else 1)"
                '''
            }
        }
        stage('Cleanup Old Images') {
            steps{
                sh '''
                    docker image prune -f
                '''
            }
        }
    }
    post {
        failure {
            sh '''
                
                echo "Rolling back to previous Docker image"

                docker rm -f devops-python-cicd-container || true
                docker run -d \
                    --name devops-python-cicd-container \
                    -p 5000:5000 \
                    v1shahid/devops-python-cicd:$PREVIOUS_BUILD

                echo "Rollback completed"
            '''
        }
    }
}