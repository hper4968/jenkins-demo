pipeline {
    agent any

    environment {
        S3_BUCKET = 'demo-hemal12'
        REMOTE_USER = 'ubuntu'
        REMOTE_HOST = '3.18.104.241'
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
                   sh "${scannerHome}/bin/sonar-scanner"
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

//    stage('Deploy') {
//     steps {
//         echo "Deploying to remote Apache server..."
//         sshagent(credentials: ['my-ssh-key-id']) {
//             sh """
//                 # Upload to a temporary path where 'ubuntu' has write access
//                 scp -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null index.html ${REMOTE_USER}@${REMOTE_HOST}:/tmp/index.html

//                 # Move it to the final destination using sudo
//                 ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null ${REMOTE_USER}@${REMOTE_HOST} 'sudo mv /tmp/index.html /var/www/html/index.html && sudo systemctl restart apache2 || sudo systemctl restart httpd'
//             """
//         }
//     }
// }

    stage('Deploy') {
        steps {
            echo "Deploying index.html from S3 to Apache server..."
            sshagent(credentials: ['my-ssh-key-id']) {
                sh """
                    ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null ${REMOTE_USER}@${REMOTE_HOST} '
                        aws s3 cp s3://${S3_BUCKET}/index.html /tmp/index.html &&
                        sudo mv /tmp/index.html /var/www/html/index.html &&
                        sudo systemctl restart apache2 || sudo systemctl restart httpd
                    '
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
