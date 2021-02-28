pipeline {
  agent {
    docker {
      image 'maven3.6.3'
    }

  }
  stages {
    stage('Build') {
      steps {
        sh 'mvn install'
      }
    }

  }
}