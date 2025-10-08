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
        - name: workspace-volume
          mountPath: /home/jenkins/agent
  volumes:
    - name: workspace-volume
      emptyDir: {}
"""
        }
    }

    environment {
        REGISTRY = "docker.io/rafikhan110"
        BACKEND_IMAGE = "backend-demo"
        FRONTEND_IMAGE = "frontend-demo"
        TAG = "latest"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/Rafy110/cicd-demo',
                        credentialsId: 'github-cred'
                    ]]
                ])
            }
        }

        stage('Build & Push Backend') {
            steps {
                container('kaniko') {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh '''
                          mkdir -p /root/.docker
                          echo "{
                            \\"auths\\": {
                              \\"https://index.docker.io/v1/\\": {
                                \\"username\\": \\"$DOCKER_USER\\",
                                \\"password\\": \\"$DOCKER_PASS\\",
                                \\"auth\\": \\"$(echo -n $DOCKER_USER:$DOCKER_PASS | base64)\\"
                              }
                            }
                          }" > /root/.docker/config.json

                          /kaniko/executor \
                            --context=${WORKSPACE}/backend \
                            --dockerfile=${WORKSPACE}/backend/Dockerfile \
                            --destination=$REGISTRY/$BACKEND_IMAGE:$TAG
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
                          mkdir -p /root/.docker
                          echo "{
                            \\"auths\\": {
                              \\"https://index.docker.io/v1/\\": {
                                \\"username\\": \\"$DOCKER_USER\\",
                                \\"password\\": \\"$DOCKER_PASS\\",
                                \\"auth\\": \\"$(echo -n $DOCKER_USER:$DOCKER_PASS | base64)\\"
                              }
                            }
                          }" > /root/.docker/config.json

                          /kaniko/executor \
                            --context=${WORKSPACE}/frontend \
                            --dockerfile=${WORKSPACE}/frontend/Dockerfile \
                            --destination=$REGISTRY/$FRONTEND_IMAGE:$TAG
                        '''
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                container('kaniko') {
                    sh '''
                      echo "Deploying backend and frontend..."
                      kubectl apply -f k8s/backend-deployment.yaml
                      kubectl apply -f k8s/frontend-deployment.yaml
                    '''
                }
            }
        }
    }
}
