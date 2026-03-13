pipeline {
    agent any

    environment {
        DOCKER_USER = "khalidali07"
        KUBECONFIG = '/var/lib/jenkins/.kube/config' // Jenkins user ke liye kubeconfig
        BACKEND_IMAGE = "khalidali07/tws-backend-threetier"
        FRONTEND_IMAGE = "khalidali07/tws-frontend-threetier"
        BACKEND_TAG = "latest"
        FRONTEND_TAG = "04"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/KhalidAli555/TWSThreeTierAppChallenge.git'
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                docker build -t $BACKEND_IMAGE:$BACKEND_TAG ./backend
                docker build -t $FRONTEND_IMAGE:$FRONTEND_TAG ./frontend
                '''
            }
        }

        stage('Login to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh '''
                    echo $PASS | docker login -u $USER --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                sh '''
                docker push $BACKEND_IMAGE:$BACKEND_TAG
                docker push $FRONTEND_IMAGE:$FRONTEND_TAG
                '''
            }
        }

        stage('Deploy Namespace') {
            steps {
                sh 'kubectl apply -f Kubernetes-Manifests-file/namespace.yaml'
            }
        }

        stage('Deploy Database') {
            steps {
                sh 'kubectl apply -f Kubernetes-Manifests-file/Database/'
            }
        }

        stage('Deploy Backend') {
            steps {
                sh 'kubectl apply -f Kubernetes-Manifests-file/Backend/'
            }
        }

        stage('Deploy Frontend') {
            steps {
                sh 'kubectl apply -f Kubernetes-Manifests-file/Frontend/'
            }
        }

        stage('Rolling Restart') {
            steps {
                sh '''
                kubectl rollout restart deployment backend -n three-tier
                kubectl rollout restart deployment frontend -n three-tier
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                kubectl get pods -n three-tier
                kubectl get svc -n three-tier
                '''
            }
        }

    }

    post {
        success {
            echo 'Deployment Successful!'
        }
        failure {
            echo 'Pipeline Failed!'
        }
    }
}
