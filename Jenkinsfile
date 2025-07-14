pipeline {
    agent any

    environment {
        S3_BUCKET = 'demo-hemal12'
        REMOTE_USER = 'ubuntu'
        REMOTE_HOST = '3.134.78.129'
        REMOTE_PATH = '/var/www/html/index.html'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'dev', url: 'https://github.com/hper4968/jenkins-demo.git'
            }
        }

        stage('Build') {
            steps {
                echo "Uploading index.html to S3..."
                sh "aws s3 cp index.html s3://${S3_BUCKET}/${S3_PATH}"
            }
        }

   stage('Sonar CodeAnalysis') {
            environment {
                scannerHome = tool 'sonar6.2' //- This gives us CLI,below code will run there,
                // tool name  (Manage Jenkins > Tools > Sonar))
            }
            steps {
               withSonarQubeEnv('sonarserver') { // server name in Jenkins Manage > system > sonarQube installation
                   sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=sonar-demo \
                   -Dsonar.projectName=sonar-demo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
              }
            }
        }

       stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploying to remote Apache server..."
                // Copy file to server using SSH credentials configured in Jenkins
                sshagent(credentials: ['my-ssh-key-id']) {
                    sh """
                        scp index.html ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_PATH}
                        ssh ${REMOTE_USER}@${REMOTE_HOST} 'sudo systemctl restart httpd'
                    """
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    def response = sh(script: "curl -o /dev/null -s -w \"%{http_code}\" http://${REMOTE_HOST}", returnStdout: true).trim()
                    echo "Apache status code: ${response}"

                    if (response != '200') {
                        error "Deployment failed! Apache returned status: ${response}"
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
