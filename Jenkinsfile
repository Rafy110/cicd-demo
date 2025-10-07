pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')   // Jenkins credentials ID
        DOCKERHUB_USER = 'rafikhan110'                     // 👈 replace with your DockerHub username
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo '📥 Cloning repository...'
                git branch: 'main', url: 'https://github.com/Rafy110/ci-cd-demo.git'
            }
        }

        stage('Build Backend Image') {
            steps {
                echo '🐳 Building backend image...'
                sh 'docker build -t $DOCKERHUB_USER/backend-demo:latest ./backend'
            }
        }

        stage('Build Frontend Image') {
            steps {
                echo '🐳 Building frontend image...'
                sh 'docker build -t $DOCKERHUB_USER/frontend-demo:latest ./frontend'
            }
        }

        stage('Push Images') {
            steps {
                echo '🚀 Pushing images to DockerHub...'
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                sh 'docker push $DOCKERHUB_USER/backend-demo:latest'
                sh 'docker push $DOCKERHUB_USER/frontend-demo:latest'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo '⚡ Deploying to Kubernetes...'
                sh 'kubectl apply -f k8s/ -n jenkins'
            }
        }
    }
}
