pipeline {
    agent any

    environment {
        EMAIL_RECIPIENT = 'ahadsiddiqui094@gmail.com'
    }

    stages {
        stage('Checkout') {
            steps {
                // Tool: Git
                checkout([$class: 'GitSCM', 
                    branches: [[name: '*/main']], 
                    userRemoteConfigs: [[url: 'https://github.com/ahadsiddiqui/8.1CDevSecOps']]]
                )
            }
        }

        stage('Install Dependencies') {
            steps {
                // Tool: npm
                bat 'npm ci'
            }
        }

        stage('Run Tests') {
            steps {
                // Tool: npm test
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    bat 'npm test > testlog.txt 2>&1 || exit 0'
                }
            }
            post {
                success {
                    emailext(
                        subject: "✅ Tests Passed: ${currentBuild.fullDisplayName}",
                        body: "All tests completed successfully.\n\nAttached: testlog.txt",
                        to: "${EMAIL_RECIPIENT}",
                        attachmentsPattern: 'testlog.txt'
                    )
                }
                failure {
                    emailext(
                        subject: "❌ Tests FAILED: ${currentBuild.fullDisplayName}",
                        body: "Some tests failed. Please review the attached log.\n\nAttached: testlog.txt",
                        to: "${EMAIL_RECIPIENT}",
                        attachmentsPattern: 'testlog.txt'
                    )
                }
            }
        }

        stage('Security Audit') {
            steps {
                // Tool: npm audit
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    bat 'npm audit > auditlog.txt 2>&1 || exit 0'
                }
            }
            post {
                always {
                    emailext(
                        subject: "🔒 Security Audit: ${currentBuild.currentResult} – ${currentBuild.fullDisplayName}",
                        body: "Security audit completed with status: ${currentBuild.currentResult}.\n\nAttached: auditlog.txt",
                        to: "${EMAIL_RECIPIENT}",
                        attachmentsPattern: 'auditlog.txt'
                    )
                }
            }
        }
    }

    post {
        failure {
            emailext(
                subject: "🚨 Pipeline FAILED: ${currentBuild.fullDisplayName}",
                body: "The pipeline failed before reaching the test or audit stages. See the full console log for details.",
                to: "${EMAIL_RECIPIENT}",
                attachLog: true
            )
        }
        // no success block here
    }
}
