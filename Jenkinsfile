pipeline { 
    agent {  docker 'maven:3-alpine'   }
    stages {
        stage ('Build') {
            steps {
                sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                    mvn -B -DskipTest=true install
                ''' 
            }
            post {
                always {
                    junit '**/target/surefire-reports/**/*.xml' 
                }
            }
        }
        stage('IQ Scan - Build') {
            steps{
                parallel(IQ: {
                    nexusPolicyEvaluation failBuildOnNetworkError: false, 
                    iqApplication: 'webgoat8', 
                    iqStage: 'build', 
                    iqScanPatterns: [[scanPattern: '']], 
                    jobCredentialsId: ''
                 },
                 OWASP: {
                    echo "Stubbed out OWASP Dep Check"
                 },
                 'Build Container': {
                    sh '''
                        cd webgoat-server
                        mvn -B docker:build
                    '''
                 })
            }

        }
        stage('Scan Container') {
            steps{
                sh "docker save mycompany.com:18444/webgoat/webgoat-8.0 -o ${env.WORKSPACE}/webgoat.tar"

                nexusPolicyEvaluation failBuildOnNetworkError: false, 
                iqApplication: 'webgoat8', 
                iqStage: 'release', 
                iqScanPatterns: [[scanPattern: '*.tar']], 
                jobCredentialsId: ''
            } 
            post {
                success {
                    sh '''
                        docker tag webgoat/webgoat-8.0 mycompany.com:18444/webgoat/webgoat-8.0:8.0
                    '''
                }
                failure {
                    echo '...the IQ Scan FAILED'
                }
            }   
        }
        stage('Publish Container') {
            steps {
                sh '''
                    docker push mycompany.com:18444/webgoat/webgoat-8.0
                '''
            }
        }
    }
}