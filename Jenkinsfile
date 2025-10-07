pipeline {
    agent {
        kubernetes {
            label 'kaniko-agent'
            defaultContainer 'jnlp'
            yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    some-label: kaniko-agent
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:latest
    command:
      - cat
    tty: true
    volumeMounts:
      - name: kaniko-secret
        mountPath: /kaniko/.docker
  volumes:
  - name: kaniko-secret
    secret:
      secretName: dockerhub-secret
"""
        }
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-cred')
        IMAGE_NAME = "rafikhan110/backend-demo:latest"
        CONTEXT_DIR = "${WORKSPACE}/backend"
        K8S_MANIFEST_DIR = "${WORKSPACE}/k8s"
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo "📥 Cloning repository..."
                checkout scm
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                container('kaniko') {
                    echo "🐳 Building & Pushing Docker image with Kaniko..."
                    sh """
                    /kaniko/executor \
                        --dockerfile=${CONTEXT_DIR}/Dockerfile \
                        --context=${CONTEXT_DIR} \
                        --destination=${IMAGE_NAME} \
                        --insecure \
                        --skip-tls-verify
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                container('jnlp') {
                    echo "🚀 Deploying to Kubernetes..."
                    sh """
                    kubectl apply -f ${K8S_MANIFEST_DIR}/
                    kubectl rollout status deployment/backend-demo
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed!"
        }
    }
}
