pipeline {
    agent any

    environment {
        // Pulls your Snyk API token from Jenkins Credentials
        SNYK_TOKEN = credentials('snyk-token')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                bat 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                bat 'npm test || exit /b 0'
            }
        }

        stage('Generate Coverage Report') {
            steps {
                bat 'npm run coverage || exit /b 0'
            }
        }

        stage('Security Scans') {
            steps {
                bat 'npm audit --json > audit.json || exit /b 0'
                bat 'snyk auth %SNYK_TOKEN%'
                bat 'snyk test --json > snyk-report.json || exit /b 0'
                bat 'type audit.json'
                bat 'type snyk-report.json'
            }
        }
    }

    post {
        always {
            emailext(
                to: 'ahadsiddiqui094@gmail.com',
                subject: "Build ${env.JOB_NAME} #${env.BUILD_NUMBER} – ${currentBuild.currentResult}",
                body: """\
Hello,

Your Jenkins job *${env.JOB_NAME}* (build #${env.BUILD_NUMBER}) has finished with status: *${currentBuild.currentResult}*.

Attached are:
- audit.json (npm audit results)
- snyk-report.json (Snyk scan report)
- Full console log

Regards,
Jenkins
""",
                attachLog: true,
                attachmentsPattern: 'audit.json,snyk-report.json'
            )
        }
    }
}
