pipeline {
  agent {
    kubernetes {
      label "test-agent"
      yaml """
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: jenkins
  containers:
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'cat']
    tty: true
"""
    }
  }
  stages {
    stage('Hello') {
      steps {
        container('busybox') {
          sh 'echo "✅ Agent pod is working"'
        }
      }
    }
  }
}
