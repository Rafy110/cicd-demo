pipeline {
  agent {
    kubernetes {
      label 'kaniko-agent'
      yaml """
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: jenkins
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    command:
      - /busybox/sh
      - -c
      - sleep 3600
    tty: true
    volumeMounts:
      - name: docker-config
        mountPath: /kaniko/.docker
  volumes:
    - name: docker-config
      emptyDir: {}
"""
    }
  }

  environment {
    REGISTRY       = "docker.io/rafikhan110"
    BACKEND_IMAGE  = "backend-demo"
    FRONTEND_IMAGE = "frontend-demo"
    TAG            = "latest"
    K8S_NAMESPACE  = "default"   // change if needed
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
          withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
            sh '''
              mkdir -p /kaniko/.docker
              AUTH=$(echo -n "$DOCKER_USER:$DOCKER_PASS" | base64 | tr -d '\\n')

              cat > /kaniko/.docker/config.json <<EOF
{
  "auths": {
    "https://index.docker.io/v1/": {
      "username": "$DOCKER_USER",
      "password": "$DOCKER_PASS",
      "auth": "$AUTH"
    }
  }
}
EOF

              /kaniko/executor \
                --context=${WORKSPACE}/backend \
                --dockerfile=${WORKSPACE}/backend/Dockerfile \
                --destination=$REGISTRY/$BACKEND_IMAGE:$TAG \
                --skip-tls-verify
            '''
          }
        }
      }
    }

    stage('Build & Push Frontend') {
      steps {
        container('kaniko') {
          withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
            sh '''
              mkdir -p /kaniko/.docker
              AUTH=$(echo -n "$DOCKER_USER:$DOCKER_PASS" | base64 | tr -d '\\n')

              cat > /kaniko/.docker/config.json <<EOF
{
  "auths": {
    "https://index.docker.io/v1/": {
      "username": "$DOCKER_USER",
      "password": "$DOCKER_PASS",
      "auth": "$AUTH"
    }
  }
}
EOF

              /kaniko/executor \
                --context=${WORKSPACE}/frontend \
                --dockerfile=${WORKSPACE}/frontend/Dockerfile \
                --destination=$REGISTRY/$FRONTEND_IMAGE:$TAG \
                --skip-tls-verify
            '''
          }
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        container('kaniko') {
          sh '''
            echo "[INFO] Installing kubectl inside Kaniko container..."
            curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
            chmod +x kubectl && mv kubectl /usr/local/bin/

            echo "[INFO] Deploying to Kubernetes namespace: $K8S_NAMESPACE"
            kubectl apply -n $K8S_NAMESPACE -f k8s/backend-deployment.yaml
            kubectl apply -n $K8S_NAMESPACE -f k8s/frontend-deployment.yaml

            echo "[INFO] Waiting for rollout..."
            kubectl rollout status -n $K8S_NAMESPACE deployment/backend-deployment --timeout=120s
            kubectl rollout status -n $K8S_NAMESPACE deployment/frontend-deployment --timeout=120s
          '''
        }
      }
    }
  }

  post {
    success {
      echo "✅ Pipeline finished successfully!"
    }
    failure {
      echo "❌ Pipeline failed, check logs!"
    }
  }
}
