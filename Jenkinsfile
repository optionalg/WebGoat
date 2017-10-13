pipeline { 
    agent any
    tools {
      maven 'M3'
      jdk 'jdk8'
    }
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
        stage('Scan App - Build Container') {
            steps{
                parallel('IQ-BOM': {
                    nexusPolicyEvaluation failBuildOnNetworkError: false, 
                    iqApplication: 'webgoat8', 
                    iqStage: 'build', 
                    iqScanPatterns: [[scanPattern: '']], 
                    jobCredentialsId: ''
                 },
                 'Static Analysis': {
                    dependencyCheckAnalyzer datadir: '', hintsFile: '', 
                    includeCsvReports: false, 
                    includeHtmlReports: false, 
                    includeJsonReports: false, 
                    isAutoupdateDisabled: false, 
                    outdir: '', 
                    scanpath: '', 
                    skipOnScmChange: false, 
                    skipOnUpstreamChange: false, 
                    suppressionFile: '', 
                    zipExtensions: ''

                    dependencyCheckPublisher canComputeNew: false, 
                    defaultEncoding: '', 
                    healthy: '', 
                    pattern: '', 
                    unHealthy: ''
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
                    error("...the IQ Scan FAILED")
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