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
  }
}