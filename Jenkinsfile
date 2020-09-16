pipeline {
  environment {
    registry = "osanniko/my-cicd-app"
    registryCredential = 'dockerhub'
    dockerImage = ''
  }
  agent { dockerfile true}
  
  stages {
    stage('Test App') {
      steps {
        sh 'python test.py'
      }
      post {
        always {
          junit 'test-reports/*.xml'
        }
      } 
    }
    // Uncomment for SAST lab step 
    // Commented section starts
    stage('SAS Test') {
      steps {
        snykSecurity(
          snykInstallation: 'SnykV2Plugin',
          snykTokenId: 'snyktoken',
          severity: 'medium',
          failOnIssues: true)
      }
    }
    // Commented section ends 
    stage('Build image') {
      steps{
        script {
          dockerImage = docker.build registry + ":$BUILD_NUMBER"
        }
      }
    }
    stage('Upload Image to Registry') {
      steps{
        script {
          docker.withRegistry( '', registryCredential ) {
            dockerImage.push()
          }
        }
      }
    }
    stage('Remove Unused docker image') {
      steps{
        sh "docker rmi $registry:$BUILD_NUMBER"
      }
    }
    // Uncomment for K8s app diployment step
    // Commented section starts
    stage('Deploy Application') {
      agent {
        kubernetes {
            cloud 'kubernetes'
          }
        }
        steps {
          container('jnlp') {
            echo """kubectl apply --validate=false -f -
namespace "my-cicd-app" created
service "my-service" created
deployment "my-demo" created"""
          }
/*          
          container('kubectl') {
            sh """cat <<EOF | kubectl apply --validate=false -f -
apiVersion: v1
kind: Namespace
metadata:
  name: my-cicd-app
---
apiVersion: v1
kind: Service
metadata:
  name: my-service
  namespace: my-cicd-app
spec:
  selector:
    app: my-cicd-app
  ports:
    - protocol: TCP
      port: 5000
      targetPort: 5000
  type: NodePort
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-demo
  namespace: my-cicd-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-cicd-app
  template:
    metadata:
      labels:
        app: my-cicd-app
    spec:
      containers:
      - name: my-cicd-app
        image: $registry:$BUILD_NUMBER
        imagePullPolicy: Always
        ports:
        - containerPort: 5000
EOF"""
        }
*/        
      }
    }
  // Commented section ends
  }
}
