pipeline {
  agent any
  stages {
    stage('Prepare') {
      steps {
        git(url: 'https://github.com/CMYanko/WebGoat.git', branch: 'develop')
      }
    }
    stage('Build') {
      steps {
        tool 'M3'
        isUnix()
        sh '\'${mvnHome}/bin/mvn\' -Dmaven.test.failure.ignore -f ./pom.xml clean package -U'
      }
    }
  }
}