pipeline {
  agent {
    kubernetes {
      label "kaniko-agent"
      yaml """
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: jenkins            # use the jenkins SA (must have pod/create permissions)
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:latest
    command: ["/busybox/sh","-c"]
    args: ["cat"]
    tty: true
    volumeMounts:
    - name: kaniko-secret
      mountPath: /kaniko/.docker
  - name: kubectl
    image: bitnami/kubectl:1.27
    command: ["/busybox/sh","-c"]
    args: ["cat"]
    tty: true
  volumes:
  - name: kaniko-secret
    secret:
      secretName: regcred
"""
    }
  }

  stages {
    stage('Prepare') {
      steps {
        script {
          // immutable tag per build
          env.IMG_TAG = "build-${env.BUILD_NUMBER}"
          echo "Image tag will be: ${env.IMG_TAG}"
        }
      }
    }

    stage('Build & Push: Backend') {
      steps {
        container('kaniko') {
          sh """
            /kaniko/executor \
              --context ${WORKSPACE}/backend \
              --dockerfile ${WORKSPACE}/backend/Dockerfile \
              --destination=rafikhan110/backend-demo:${IMG_TAG} \
              --cache=true
          """
        }
      }
    }

    stage('Build & Push: Frontend') {
      steps {
        container('kaniko') {
          sh """
            /kaniko/executor \
              --context ${WORKSPACE}/frontend \
              --dockerfile ${WORKSPACE}/frontend/Dockerfile \
              --destination=rafikhan110/frontend-demo:${IMG_TAG} \
              --cache=true
          """
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        container('kubectl') {
          // Ensure manifests exist (create/update)
          sh "kubectl apply -f k8s/ -n jenkins"

          // Update deployments to the images we just pushed
          sh "kubectl set image deployment/backend-deployment backend=rafikhan110/backend-demo:${IMG_TAG} -n jenkins"
          sh "kubectl set image deployment/frontend-deployment frontend=rafikhan110/frontend-demo:${IMG_TAG} -n jenkins"

          // Wait for rollouts
          sh "kubectl rollout status deployment/backend-deployment -n jenkins --timeout=120s"
          sh "kubectl rollout status deployment/frontend-deployment -n jenkins --timeout=120s"
        }
      }
    }
  }

  post {
    failure {
      echo "Pipeline failed — check console output"
    }
    success {
      echo "Pipeline succeeded — deployed tag ${IMG_TAG}"
    }
  }
}
