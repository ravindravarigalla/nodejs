pipeline {

  environment {
    PROJECT = " still-smithy-279711"
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
  - name: docker
    image: docker:stable
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
    stage('build') {
      steps {
        container('nodejs') {
          sh "#npm install"
          sh "#npm test"
        }
      }
    }
    
    stage('Build and push image with Container Builder') {
      steps {
        container('docker') {
          sh "gcloud auth list"
          sh "#chmod root:docker /var/run/docker.sock"
          sh "docker build -t gg ."
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
          helm install sampleapp3 sampleapp/ --namespace default
          helm ls
          kubectl get pods --namespace default
          """ 
        }
      }
    }
  }
}

