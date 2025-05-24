pipeline {
    agent any

    environment {
        // Email recipient for notifications
        EMAIL_RECIPIENT = 'ahadsiddiqui094@gmail.com'
    }

    stages {
        stage('Checkout') {
            steps {
                // Tool: Git (via Jenkins Git plugin)
                // Purpose: Fetch the latest code from GitHub
                checkout([$class: 'GitSCM', 
                    branches: [[name: '*/main']], 
                    userRemoteConfigs: [[url: 'https://github.com/ahadsiddiqui/8.1CDevSecOps']]
                ])
            }
        }

        stage('Install Dependencies') {
            steps {
                // Tool: npm (Node Package Manager)
                // Purpose: Install all project dependencies
                bat 'npm ci'
            }
        }

        stage('Run Tests') {
            steps {
                // Tool: npm test (likely using Mocha or Jest)
                // Purpose: Execute unit tests and capture the output
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    bat 'npm test > testlog.txt || exit 0'
                }
            }
        }

        stage('Security Audit') {
            steps {
                // Tool: npm audit
                // Purpose: Scan for security vulnerabilities in dependencies
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    bat 'npm audit > auditlog.txt || exit 0'
                }
            }
        }
    }

    post {
        // Tool: Email Extension Plugin
        // Purpose: Send pipeline success email with logs
        success {
            emailext subject: "Pipeline Successful: ${currentBuild.fullDisplayName}",
                     body: "The pipeline completed successfully. Logs are attached.",
                     to: "${EMAIL_RECIPIENT}",
                     attachmentsPattern: "**/testlog.txt, **/auditlog.txt",
                     attachLog: true
        }

        // Tool: Email Extension Plugin
        // Purpose: Send pipeline failure email with logs
        failure {
            emailext subject: "Pipeline Failed: ${currentBuild.fullDisplayName}",
                     body: "The pipeline failed. Logs are attached for review.",
                     to: "${EMAIL_RECIPIENT}",
                     attachmentsPattern: "**/testlog.txt, **/auditlog.txt",
                     attachLog: true
        }
    }
}
