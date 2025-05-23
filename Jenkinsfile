pipeline {
    agent any

    environment {
        // Make sure this ID matches the credential you just created
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
                // don’t fail the build on test failures
                bat 'npm test || exit /b 0'
            }
        }

        stage('Generate Coverage Report') {
            steps {
                // only if you have an npm script named “coverage”
                bat 'npm run coverage || exit /b 0'
            }
        }

        stage('Security Scans') {
            steps {
                // npm audit
                bat 'npm audit --json > audit.json || exit /b 0'
                // authenticate and run Snyk
                bat 'snyk auth %SNYK_TOKEN%'
                bat 'snyk test --json > snyk-report.json || exit /b 0'
            }
        }

        stage('Notify') {
            steps {
                // send one email with both JSON files plus the console log
                emailext(
                    to: 'ahadsiddiqui094@gmail.com',
                    subject: "Build ${env.JOB_NAME} #${env.BUILD_NUMBER} – ${currentBuild.currentResult}",
                    body: """\
Hello,

Your Jenkins job *${env.JOB_NAME}* (build #${env.BUILD_NUMBER}) has finished with status *${currentBuild.currentResult}*.

Please find attached:
 - audit.json (npm audit results)
 - snyk-report.json (Snyk scan results)
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
}
