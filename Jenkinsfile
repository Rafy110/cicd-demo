pipeline {
  agent {
    kubernetes {
      label "docker-agent"
      yaml """
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: jenkins
  containers:
  - name: docker
    image: docker:24.0.7-dind
    securityContext:
      privileged: true
    command:
    - dockerd-entrypoint.sh
    args:
    - --host=tcp://0.0.0.0:2375
    - --host=unix:///var/run/docker.sock
    volumeMounts:
    - mountPath: /certs/client
      name: docker-certs
  - name: docker-cli
    image: docker:24.0.7-cli
    command:
    - sleep
    args:
    - "9999"
    volumeMounts:
    - mountPath: /certs/client
      name: docker-certs
  volumes:
  - name: docker-certs
    emptyDir: {}
"""
    }
  }

  environment {
    DOCKER_HOST = "tcp://localhost:2375"
    IMAGE_NAME = "rafikhan110/cicd-demo:latest"
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main',
            url: 'https://github.com/Rafy110/cicd-demo.git',
            credentialsId: 'github-cred'
      }
    }

    stage('Build Docker Image') {
      steps {
        container('docker-cli') {
          sh """
            docker build -t ${IMAGE_NAME} ./backend
            docker build -t rafikhan110/frontend-demo:latest ./frontend
          """
        }
      }
    }

    stage('Push to Docker Hub') {
      steps {
        container('docker-cli') {
          withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
            sh """
              echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
              docker push ${IMAGE_NAME}
              docker push rafikhan110/frontend-demo:latest
            """
          }
        }
      }
    }

    stage('Deploy to Minikube') {
      steps {
        sh """
          kubectl apply -f k8s/deployment.yaml
          kubectl rollout status deployment/cicd-demo
        """
      }
    }
  }
}
