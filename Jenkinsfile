pipeline {
  agent any
  stages {
    stage('Prepare') {
      steps {
        git(url: 'https://github.com/CMYanko/WebGoat.git', branch: 'develop')
        tool 'M3'
        tool 'JDK8'
      }
    }
    stage('Build') {
      steps {
        isUnix()
        sh '${mvnHome}/bin/mvn\' -Dmaven.test.failure.ignore -f ./webgoat/pom.xml clean install -U'
      }
    }
  }
}