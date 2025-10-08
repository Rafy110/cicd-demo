pipeline {
    agent {
        kubernetes {
            label 'kaniko-agent'
            yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: jenkins-kaniko
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    command:
    - cat
    tty: true
    volumeMounts:
    - name: workspace-volume
      mountPath: /home/jenkins/agent
  volumes:
  - name: workspace-volume
    emptyDir: {}
"""
        }
    }

    environment {
        DOCKER_CREDENTIALS = credentials('dockerhub') // Jenkins DockerHub credentials (ID = dockerhub)
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
                        sh """
                        mkdir -p /root/.docker
                        echo '{
                          "auths": {
                            "https://index.docker.io/v1/": {
                              "username": "${DOCKER_USER}",
                              "password": "${DOCKER_PASS}",
                              "auth": "$(echo -n ${DOCKER_USER}:${DOCKER_PASS} | base64)"
                            }
                          }
                        }' > /root/.docker/config.json

                        /kaniko/executor \
                          --context=\$(pwd)/backend \
                          --dockerfile=\$(pwd)/backend/Dockerfile \
                          --destination=docker.io/${DOCKER_USER}/backend-demo:latest
                        """
                    }
                }
            }
        }

        stage('Build & Push Frontend') {
            steps {
                container('kaniko') {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                        /kaniko/executor \
                          --context=\$(pwd)/frontend \
                          --dockerfile=\$(pwd)/frontend/Dockerfile \
                          --destination=docker.io/${DOCKER_USER}/frontend-demo:latest
                        """
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                container('kaniko') {
                    sh """
                    # Install kubectl inside kaniko container
                    curl -LO "https://dl.k8s.io/release/$(curl -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                    chmod +x kubectl
                    mv kubectl /usr/local/bin/

                    # Apply Kubernetes manifests
                    kubectl apply -f k8s/backend-deployment.yaml
                    kubectl apply -f k8s/frontend-deployment.yaml
                    """
                }
            }
        }
    }
}
