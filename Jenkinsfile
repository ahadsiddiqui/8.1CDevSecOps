pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                // Pull Jenkinsfile + code
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                // Windows: install npm packages
                bat 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                // Continue even if tests fail
                bat 'npm test || exit /b 0'
            }
        }

        stage('Generate Coverage') {
            steps {
                // Only if you have a "coverage" script
                bat 'npm run coverage || exit /b 0'
            }
        }

        stage('Security Scans') {
            steps {
                // 1) npm audit
                bat 'npm audit --json > audit.json || exit /b 0'

                // 2) Snyk authenticated scan
                script {
                    withCredentials([string(credentialsId: 'snyk-token', variable: 'SNYK_TOKEN')]) {
                        bat 'snyk auth %SNYK_TOKEN%'
                        bat 'snyk test --json > snyk-report.json || exit /b 0'
                    }
                }

                // 3) Show summaries in console
                bat 'type audit.json'
                bat 'type snyk-report.json'
            }
        }

        stage('Notify') {
            steps {
                // Send email with both reports + full log
                emailext(
                    to: 'ahadsiddiqui094@gmail.com',
                    subject: "Build ${env.JOB_NAME} #${env.BUILD_NUMBER} – ${currentBuild.currentResult}",
                    body: """\
Hello,

Your Jenkins job *${env.JOB_NAME}* (build #${env.BUILD_NUMBER}) has finished with status *${currentBuild.currentResult}*.

Attached:
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
}
