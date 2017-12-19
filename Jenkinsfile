pipeline { 
    agent any
    tools {
      maven 'M3'
      jdk 'jdk8'
    }

    def postGitHub(commitId, state, context, description, targetUrl) {
    def payload = JsonOutput.toJson(
    state: state,
    context: context,
    description: description,
    target_url: targetUrl
    )
        sh "curl -H \"Authorization: token ${gitHubApiToken}\" --request POST --data '${payload}'
        https://api.github.com/repos/${project}/statuses/${commitId} > /dev/null"
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
                failure {
                    postGitHub commitId, 'success', 'build', 'Build FAILED'
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
                    echo '...run SonarQube or other SAST tools here'
                 },
                 'Build Container': {
                    sh '''
                        cd webgoat-server
                        mvn -B docker:build
                    '''
                 })
            }

        }
        stage('Test Container') {
            steps{
                echo '...run container and test it'
            } 
            post {
                success {
                    echo '...the Test Scan Passed!'
                }
                failure {
                    echo '...the Test  FAILED'
                    error("...the Container Test FAILED")
                }
            }   
        }
        stage('Scan Container') {
            steps{
                sh "docker save webgoat/webgoat-8.0 -o ${env.WORKSPACE}/webgoat.tar"

                nexusPolicyEvaluation failBuildOnNetworkError: false, 
                iqApplication: 'webgoat8', 
                iqStage: 'release', 
                iqScanPatterns: [[scanPattern: '*.tar']], 
                jobCredentialsId: ''
            } 
            post {
                success {
                    postGitHub commitId, 'failure', 'build', 'IQ Scan PASSED'
                }
                failure {
                    postGitHub commitId, 'failure', 'build', 'IQ Scan FAILED'
                    error("...the IQ Scan FAILED")
                }
            }   
        }
        stage('Publish Container') {
            when {
                branch 'master'
            }
            steps {
                sh '''
                    docker tag webgoat/webgoat-8.0 mycompany.com:5000/webgoat/webgoat-8.0:8.0
                    docker push mycompany.com:5000/webgoat/webgoat-8.0
                '''
            }
        }
    }
}