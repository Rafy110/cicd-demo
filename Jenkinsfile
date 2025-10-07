pipeline {
  agent {
    kubernetes {
      label "kaniko-agent"
      yaml """
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: jenkins
  containers:
    - name: kaniko
      image: gcr.io/kaniko-project/executor:v1.9.0-debug
      command: ["cat"]
      tty: true
    - name: kubectl
      image: bitnami/kubectl:1.29
      command: ["cat"]
      tty: true

"""
    }
  }

  environment {
    DOCKERHUB_USER = "rafikhan110"      // your DockerHub username
    DOCKERHUB_REPO = "cicd-demo"        // repo name
    BACKEND_IMAGE = "backend"
    FRONTEND_IMAGE = "frontend"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build & Push Backend Image') {
      steps {
        container('kaniko') {
          sh '''
            /kaniko/executor \
              --context ./backend \
              --dockerfile ./backend/Dockerfile \
              --destination=${DOCKERHUB_USER}/${DOCKERHUB_REPO}:${BACKEND_IMAGE}-latest
          '''
        }
      }
    }

    stage('Build & Push Frontend Image') {
      steps {
        container('kaniko') {
          sh '''
            /kaniko/executor \
              --context ./frontend \
              --dockerfile ./frontend/Dockerfile \
              --destination=${DOCKERHUB_USER}/${DOCKERHUB_REPO}:${FRONTEND_IMAGE}-latest
          '''
        }
      }
    }

    stage('Deploy to Minikube') {
      steps {
        container('kubectl') {
          sh '''
            kubectl apply -f k8s/backend-deployment.yaml
            kubectl apply -f k8s/frontend-deployment.yaml
            kubectl rollout status deployment/backend -n default
            kubectl rollout status deployment/frontend -n default
          '''
        }
      }
    }
  }
}
