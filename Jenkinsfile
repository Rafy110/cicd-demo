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
    app: jenkins-kaniko
spec:
  serviceAccountName: jenkins
  containers:
  - name: jnlp
    image: jenkins/inbound-agent:latest
    args: ['\$(JENKINS_SECRET)', '\$(JENKINS_NAME)']
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    command:
      - /busybox/sh
      - -c
      - "sleep 3600"
    tty: true
"""
    }
  }

  environment {
    REGISTRY = "docker.io"
    DOCKERHUB_REPO = "rafikhan110"
    BACKEND_IMAGE = "backend-demo"
    FRONTEND_IMAGE = "frontend-demo"
    TAG = "latest"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build & Push Backend') {
      steps {
        container('kaniko') {
          withCredentials([usernamePassword(credentialsId: 'dockerhub',
                                           usernameVariable: 'DOCKER_USER',
                                           passwordVariable: 'DOCKER_PASS')]) {
            sh """
              mkdir -p /kaniko/.docker/
              echo '{
                "auths": {
                  "https://index.docker.io/v1/": {
                    "username": "${DOCKER_USER}",
                    "password": "${DOCKER_PASS}",
                    "auth": "'\$(echo -n ${DOCKER_USER}:${DOCKER_PASS} | base64)'"
                  }
                }
              }' > /kaniko/.docker/config.json

              /kaniko/executor \
                --context=\${WORKSPACE}/backend \
                --dockerfile=\${WORKSPACE}/backend/Dockerfile \
                --destination=\${REGISTRY}/\${DOCKERHUB_REPO}/\${BACKEND_IMAGE}:\${TAG} \
                --docker-config=/kaniko/.docker/
            """
          }
        }
      }
    }

    stage('Build & Push Frontend') {
      steps {
        container('kaniko') {
          withCredentials([usernamePassword(credentialsId: 'dockerhub-cred',
                                           usernameVariable: 'DOCKER_USER',
                                           passwordVariable: 'DOCKER_PASS')]) {
            sh """
              mkdir -p /kaniko/.docker/
              echo '{
                "auths": {
                  "https://index.docker.io/v1/": {
                    "username": "${DOCKER_USER}",
                    "password": "${DOCKER_PASS}",
                    "auth": "'\$(echo -n ${DOCKER_USER}:${DOCKER_PASS} | base64)'"
                  }
                }
              }' > /kaniko/.docker/config.json

              /kaniko/executor \
                --context=\${WORKSPACE}/frontend \
                --dockerfile=\${WORKSPACE}/frontend/Dockerfile \
                --destination=\${REGISTRY}/\${DOCKERHUB_REPO}/\${FRONTEND_IMAGE}:\${TAG} \
                --docker-config=/kaniko/.docker/
            """
          }
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        container('jnlp') {
          sh """
            kubectl apply -f k8s/backend-deployment.yaml
            kubectl apply -f k8s/frontend-deployment.yaml
          """
        }
      }
    }
  }
}
