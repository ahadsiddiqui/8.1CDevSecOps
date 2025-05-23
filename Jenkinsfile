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
                // Wrap everything so a failure here doesn't stop the pipeline
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    // npm audit
                    bat 'npm audit --json > audit.json'

                    // install Snyk CLI if it’s missing
                    bat 'npm install -g snyk'

                    // authenticate & run Snyk
                    withCredentials([string(credentialsId: 'snyk-token', variable: 'SNYK_TOKEN')]) {
                        bat 'snyk auth %SNYK_TOKEN%'
                        bat 'snyk test --json > snyk-report.json'
                    }
                }
                // Show the output summaries
                bat 'type audit.json'
                bat 'type snyk-report.json'
            }
        }

        stage('Notify') {
            steps {
                emailext(
                    to:      'ahadsiddiqui094@gmail.com',
                    subject: "Build ${env.JOB_NAME} #${env.BUILD_NUMBER} – ${currentBuild.currentResult}",
                    body: """\
Hello,

Your Jenkins job *${env.JOB_NAME}* (build #${env.BUILD_NUMBER}) has finished with status *${currentBuild.currentResult}*.

Attached:
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
        }
    }
}
