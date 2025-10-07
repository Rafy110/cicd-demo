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
    image: gcr.io/kaniko-project/executor:latest
    command:
    - cat
    tty: true
    volumeMounts:
    - name: docker-config
      mountPath: /kaniko/.docker
  - name: kubectl
    image: bitnami/kubectl:latest
    command:
    - sh
    - -c
    - sleep 9999
    tty: true
  volumes:
  - name: docker-config
    secret:
      secretName: regcred
"""
    }
  }

  environment {
    IMAGE_NAME = "rafy110/cicd-demo:latest"
  }

  stages {
    stage('Checkout Code') {
      steps {
        git branch: 'main', 
            url: 'https://github.com/Rafy110/cicd-demo.git',
            credentialsId: 'github-cred'
      }
    }

    stage('Build and Push Image with Kaniko') {
      steps {
        container('kaniko') {
          sh """
            /kaniko/executor \
              --context `pwd` \
              --dockerfile `pwd`/Dockerfile \
              --destination=${IMAGE_NAME}
          """
        }
      }
    }

    stage('Deploy to Minikube') {
      steps {
        container('kubectl') {
          sh """
            kubectl apply -f k8s/deployment.yaml
            kubectl rollout status deployment/cicd-demo
          """
        }
      }
    }
  }
}
