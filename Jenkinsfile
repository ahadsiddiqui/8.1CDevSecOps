pipeline {
    agent any

    environment {
        EMAIL_RECIPIENT = 'ahadsiddiqui094@gmail.com'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: 'https://github.com/ahadsiddiqui/8.1CDevSecOps']]
                ])
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
            post {
                success {
                    emailext(
                        subject: "Tests Passed - ${env.JOB_NAME} [${env.BUILD_NUMBER}]",
                        body: "The test stage completed successfully.\n\nPlease find the log attached.",
                        to: "${EMAIL_RECIPIENT}",
                        attachmentsPattern: 'testlog.txt'
                    )
                }
                failure {
                    emailext(
                        subject: "Tests Failed - ${env.JOB_NAME} [${env.BUILD_NUMBER}]",
                        body: "The test stage failed.\n\nPlease review the attached log for details.",
                        to: "${EMAIL_RECIPIENT}",
                        attachmentsPattern: 'testlog.txt'
                    )
                }
            }
        }

        stage('Security Audit') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    bat 'npm audit > auditlog.txt'
                }
            }
            post {
                success {
                    emailext(
                        subject: "Security Scan Passed - ${env.JOB_NAME} [${env.BUILD_NUMBER}]",
                        body: "Security scan completed with no critical issues.\n\nLog attached.",
                        to: "${EMAIL_RECIPIENT}",
                        attachmentsPattern: 'auditlog.txt'
                    )
                }
                failure {
                    emailext(
                        subject: "Security Issues Detected - ${env.JOB_NAME} [${env.BUILD_NUMBER}]",
                        body: "Security vulnerabilities were found.\n\nSee the attached audit report.",
                        to: "${EMAIL_RECIPIENT}",
                        attachmentsPattern: 'auditlog.txt'
                    )
                }
            }
        }
    }

    post {
        success {
            emailext(
                subject: "Pipeline Successful - ${env.JOB_NAME} [${env.BUILD_NUMBER}]",
                body: "The pipeline executed successfully. All stages completed without error.",
                to: "${EMAIL_RECIPIENT}",
                attachLog: true
            )
        }
        failure {
            emailext(
                subject: "Pipeline Failed - ${env.JOB_NAME} [${env.BUILD_NUMBER}]",
                body: "One or more stages failed in the pipeline.\n\nSee Jenkins logs for details.",
                to: "${EMAIL_RECIPIENT}",
                attachLog: true
            )
        }
    }
}
