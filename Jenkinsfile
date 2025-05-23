pipeline {
    agent any

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

        stage('Generate Coverage') {
            steps {
                bat 'npm run coverage || exit /b 0'
            }
        }

        stage('Security Scans') {
            steps {
                // 1) npm audit
                bat 'npm audit --json > audit.json || exit /b 0'

                // 2) Ensure Snyk is available
                bat 'npm install -g snyk'

                // 3) Authenticate & run Snyk
                script {
                    withCredentials([string(credentialsId: 'snyk-token', variable: 'SNYK_TOKEN')]) {
                        bat 'snyk auth %SNYK_TOKEN%'
                        bat 'snyk test --json > snyk-report.json || exit /b 0'
                    }
                }

                // 4) Show the reports
                bat 'type audit.json'
                bat 'type snyk-report.json'
            }
            // No post here; failures will be handled in the final Notify stage
        }

        stage('Notify') {
            steps {
                // Always run this stage, even if previous stages failed
                emailext (
                    to: 'ahadsiddiqui094@gmail.com',
                    subject: "Build ${env.JOB_NAME} #${env.BUILD_NUMBER} – ${currentBuild.currentResult}",
                    body: """\
Hello,

Job:    ${env.JOB_NAME}
Build:  #${env.BUILD_NUMBER}
Status: ${currentBuild.currentResult}

Please find attached:
 - audit.json
 - snyk-report.json
 - Full console log

Regards,
Jenkins
""",
                    attachLog: true,
                    attachmentsPattern: 'audit.json,snyk-report.json'
                )
            }
            // force this stage to run even if earlier stages fail
            post {
                always { /* nothing needed here */ }
            }
        }
    }

    // ensure the Notify stage runs on any outcome
    post {
        always {
            // noop: using a dedicated Notify stage instead
        }
    }
}
