pipeline {
    agent any

    environment {
        NAMESPACE = "chat-app"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/MOUNTHOG/FullStack-Chat-App.git'
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                    docker build -t chat-backend ./backend
                    docker build -t chat-frontend ./frontend
                '''
            }
        }

        stage('Load Images into Kind') {
            steps {
                sh '''
                    kind load docker-image chat-backend
                    kind load docker-image chat-frontend
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl apply -f k8s/
                '''
            }
        }

        stage('Wait for Pods') {
            steps {
                sh '''
                    kubectl wait --for=condition=ready pod -l app=backend -n $NAMESPACE --timeout=120s
                    kubectl wait --for=condition=ready pod -l app=frontend -n $NAMESPACE --timeout=120s
                '''
            }
        }

        stage('Test Application') {
            steps {
                sh '''
                    kubectl port-forward svc/frontend-svc 3000:80 -n $NAMESPACE &
                    sleep 10
                    curl -f http://localhost:3000 || exit 1
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline executed successfully!'
        }
        failure {
            echo '❌ Pipeline failed!'
        }
    }
}