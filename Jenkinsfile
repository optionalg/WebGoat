pipeline {
  agent any
  def mvnHome
  env.JAVA_HOME="{tool'JDK8'}"
  env.PATH="${env.JAVA_HOME}/bin:${env.PATH}"

  stages {
    stage('Prepare') {
      steps {
        git(url: 'https://github.com/CMYanko/WebGoat.git', branch: 'develop')
      }
    }
    stage('Build') {
      steps {
        mvnHome = tool 'M3'
        isUnix()
        sh "'${mvnHome}/bin/mvn\' -Dmaven.test.failure.ignore -f ./pom.xml clean package -U"
      }
    }
  }
}