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
                sh "aws s3 cp index.html s3://${S3_BUCKET}/"
            }
        }

        stage('Sonar Code Analysis') {
            environment {
                scannerHome = tool 'sonar6.2'  // Make sure this name matches what's in Jenkins (Manage Jenkins > Tools > SonarQube Scanner)
            }
            steps {
                withSonarQubeEnv('sonarserver') { // Make sure this matches the SonarQube installation name under Manage Jenkins > Configure System
                   sh '''${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=sonar-demo \
                        -Dsonar.projectName=sonar-demo \
                        -Dsonar.sources=. \
                        -Dsonar.inclusions=index.html \
                        -Dsonar.language=web'''

                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

    stage('Deploy') {
        steps {
            echo "Deploying to remote Apache server..."
            sshagent(credentials: ['my-ssh-key-id']) {
                sh """
                    scp -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null index.html ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_PATH}
                    ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null ${REMOTE_USER}@${REMOTE_HOST} 'sudo systemctl restart apache2 || sudo systemctl restart httpd'
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
