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
    - /busybox/sh
    - -c
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
    - cat
    tty: true
  volumes:
  - name: docker-config
    secret:
      secretName: regcred
"""
    }
  }
  stages {
    stage('Build Image with Kaniko') {
      steps {
        container('kaniko') {
          sh '''
            /kaniko/executor \
              --context `pwd` \
              --dockerfile `pwd`/Dockerfile \
              --destination=rafy110/cicd-demo:latest
          '''
        }
      }
    }

    stage('Deploy to Minikube') {
      steps {
        container('kubectl') {
          sh '''
            kubectl apply -f k8s/deployment.yaml
            kubectl rollout status deployment/cicd-demo
          '''
        }
      }
    }
  }
}
