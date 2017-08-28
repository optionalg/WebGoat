pipeline { 
    agent any
    tools {
      maven 'M3'
      jdk 'jdk8'
    }
    stages {
        stage ('Pre-Build') {
            steps {
                sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                ''' 
            }
            steps {
                // Run the maven build
                if (isUnix()) {
                    sh "'${mvnHome}/bin/mvn' -B help:effective-pom"
                } else {
                    bat(/"${mvnHome}\bin\mvn" -B help:effective-pom/)
                }
            }
        }  
        stage('Build') { 
             steps {
                sh 'mvn -B -Dmaven.test.failure.ignore=true clean install' 
            }
        }
    }
}