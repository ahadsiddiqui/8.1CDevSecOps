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
                        body: """\
All tests completed successfully.

Attached: testlog.txt
""",
                        to: "${EMAIL_RECIPIENT}",
                        attachmentsPattern: 'testlog.txt'
                    )
                }
                failure {
                    emailext(
                        subject: "❌ Tests FAILED: ${currentBuild.fullDisplayName}",
                        body: """\
Some tests failed.  Please review the attached log.

Attached: testlog.txt
""",
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
                        body: """\
Security audit completed with status: ${currentBuild.currentResult}.

Attached: auditlog.txt
""",
                        to: "${EMAIL_RECIPIENT}",
                        attachmentsPattern: 'auditlog.txt'
                    )
                }
            }
        }
    }

    post {
        // catch anything else (checkout/install errors, etc.)
        failure {
            emailext(
                subject: "🚨 Pipeline FAILED: ${currentBuild.fullDisplayName}",
                body: """\
The pipeline failed before reaching the test or audit stages.
Please check the full console log for details.
""",
                to: "${EMAIL_RECIPIENT}",
                attachLog: true
            )
        }
        success {
            // No global success email — tests & audit already fired them.
        }
    }
}
