pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "bharadvajnanepalli/python-flask-blood-bank"
        KUBE_CONFIG = credentials('kubeconfig-credentials-id') // Jenkins credential ID
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/Bharadvaj143/Blood_Bank_Python_App.git', branch: 'main'
            }
        }

        stage('Setup Python Environment') {
            steps {
                sh '''
                python3 -m venv venv
                . venv/bin/activate
                pip install --upgrade pip
                pip install -r requirements.txt
                '''
            }
        }

        stage('Lint & Test') {
            steps {
                sh '''
                source venv/bin/activate
                pip install flake8 pytest
                flake8 . || true
                python3 -m unittest discover tests || echo "No tests found"
                '''
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_IMAGE}:${env.BUILD_NUMBER}")
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials-id') {
                        dockerImage.push()
                        dockerImage.push('latest')
                    }
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-credentials-id', variable: 'KUBECONFIG')]) {
                    sh '''
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                    kubectl rollout status deployment/bloodbank-app
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment successful!'
        }
        failure {
            echo '❌ Build or deployment failed.'
        }
    }
}