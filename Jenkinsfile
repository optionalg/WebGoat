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
        }  
    stage('Build') { 
        steps {
            sh 'mvn -B -Dmaven.test.failure.ignore=true clean install' 
        }
        post {
            always {
                junit '**/target/surefire-reports/**/*.xml' 
            }
        }
    }
    stage('IQ Scan - Build') {
            steps{
                nexusPolicyEvaluation failBuildOnNetworkError: false, iqApplication: 'webgoat8', iqStage: 'build', iqScanPatterns: [[scanPattern: '']], jobCredentialsId: '6f9e8ba7-b926-4ce1-b83f-f9c203c955e8'
            }
        }
    }
}