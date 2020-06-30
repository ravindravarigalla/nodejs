pipeline {

  environment {
    PROJECT = "	still-smithy-279711"
    APP_NAME = "sample"
    FE_SVC_NAME = "${APP_NAME}"
    CLUSTER = "cluster-1"
    CLUSTER_ZONE = "us-central1-c"
    IMAGE_TAG = "gcr.io/${PROJECT}/${APP_NAME}:latest"
    JENKINS_CRED = "${PROJECT}"
  }

  agent {
    kubernetes {
      
      defaultContainer 'jnlp'
      yaml """
apiVersion: v1
kind: Pod
metadata:
labels:
  component: ci
spec:
  # Use service account that can deploy to all namespaces
  
  containers:
  - name: nodejs
    image: node:10.11.0-alpine
    command:
    - cat
    tty: true
  - name: gcloud
    image: gcr.io/google.com/cloudsdktool/cloud-sdk:latest
    command:
    - cat
    tty: true
  - name: helm
    image: us.gcr.io/still-smithy-279711/helm3
    command:
    - cat
    tty: true
    
"""
}
  }
  stages {
    stage('Test') {
      steps {
        container('nodejs') {
          sh "npm install"
          sh "npm test"
        }
      }
    }
    stage('Build and push image with Container Builder') {
      steps {
        container('gcloud') {
          sh "gcloud auth list"
          sh  "apt-get update"
          sh " apt-get install \
              apt-transport-https \
              ca-certificates \
              curl \
              gnupg-agent \
              software-properties-common"
          sh" curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -"
          sh "add-apt-repository \
              deb [arch=amd64] https://download.docker.com/linux/debian \
               $(lsb_release -cs) \
               stable" "
          sh "apt-get update"
          sh "apt-get install docker-ce docker-ce-cli containerd.io"
          sh "docker build -t my-image . " 
          sh "PYTHONUNBUFFERED=1 gcloud builds submit -t  us.gcr.io/still-smithy-279711/nodejs . "
        }
      }
    }
    stage('Deploy ') {
      steps {
        container('helm') {
          sh """
          #helm ls
          gcloud container clusters get-credentials cluster-1 --zone us-central1-c --project still-smithy-279711
          kubectl get pods --namespace default
          helm repo add stable https://kubernetes-charts.storage.googleapis.com/ 
          helm repo update 
          helm install sampleapp2 sampleapp/ --namespace default
          helm ls
          kubectl get pods --namespace default
          """ 
        }
      }
    }
  }
}
