pipeline {
    agent { label 'docker' }

    environment {
        DOCKERHUB_USER = "YOUR_DOCKERHUB_USERNAME"
        IMAGE = "docker.io/${DOCKERHUB_USER}/demo-app"
        TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Build & Test') {
            steps {
                sh '''
                pip3 install -r requirements.txt
                pytest
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t $IMAGE:$TAG ."
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh '''
                    echo $PASS | docker login -u $USER --password-stdin
                    docker push $IMAGE:$TAG
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                sed -i "s|IMAGE_TAG|$TAG|g" k8s-deployment.yaml
                kubectl apply -f k8s-deployment.yaml
                '''
            }
        }
    }
}
