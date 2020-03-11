pipeline {
  agent any

  stages {
    stage('Build result') {
      steps {
        sh 'docker build -t chilv/result ./result'
      }
    } 
    stage('Build vote') {
      steps {
        sh 'docker build -t chilv/vote ./vote'
      }
    }
    stage('Build worker') {
      steps {
        sh 'docker build -t chilv/worker ./worker'
      }
    }
    stage ('login dockerhub'){
      steps{
        sh 'echo ** Logging in ***'
        sh 'docker login -u chilv -p "toiyeuem1990"'
      }
    }
    stage('Push result image') {
      steps {
        sh 'docker push chilv/result'
      }
    }
    stage('Push vote image') {
      steps {
        sh 'docker push chilv/vote'
      }
    }
    stage('Push worker image') {
      steps {
        sh 'docker push chilv/worker'
      }
    }
    stage('Deoloy to k8s') {
      steps {
        sh 'scp -i /opt/key/ssh /var/lib/jenkins/workspace/vote-app-CI/k8s/* admin@3.87.182.183:/tmp/'
        sh 'ssh -i /opt/key/ssh admin@3.87.182.183 "cd /tmp && kubectl delete -f worker-app-deployment.yml"'
      }
    }
  }
}