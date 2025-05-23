pipeline {
    agent any

    environment {
        EMAIL_RECIPIENT = 'ahadsiddiqui094@gmail.com'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                bat 'npm ci'
            }
        }

        stage('Run Tests') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    bat 'npm test > testlog.txt'
                }
            }
        }

        stage('Security Audit') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                    bat 'npm audit > auditlog.txt'
                }
            }
        }
    }

    post {
        always {
            script {
                emailext(
                    subject: "Jenkins Build: ${currentBuild.currentResult} - ${env.JOB_NAME}",
                    body: """Hi Ahad,

The Jenkins build for job '${env.JOB_NAME}' has finished with status: ${currentBuild.currentResult}.

You can view the build at: ${env.BUILD_URL}

Regards,
Jenkins""",
                    to: "${EMAIL_RECIPIENT}",
                    attachmentsPattern: '**/testlog.txt, **/auditlog.txt',
                    mimeType: 'text/plain'
                )
            }
        }
    }
}
