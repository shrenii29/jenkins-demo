pipeline {
  agent any
  stages {
    stage ('Clone Repository') {
      steps {
        echo 'Cloning Repo...'
        checkout scm
      }
    }
    stage ('Build') {
      steps {
        echo 'Building the project'
        echo 'Building complete'
      }
    }
    stage ('Test') {
      steps {
        echo 'Testing model'
        echo 'Testing Complete'
      }
    }
    stage ('Deploy') {
      steps {
        echo 'Deploying...'
        echo 'Deployment Complete'
      }
    }
  }
   post {
      success {
        echo 'Pipeline successfully executed'
      }
      failure {
        echo 'Pipeline failed'
      }
   }
}
