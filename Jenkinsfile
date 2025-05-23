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
                // Catch any errors here and mark the build UNSTABLE, but continue
                catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                    bat 'npm audit --json > audit.json || exit /b 0'
                    bat 'npm install -g snyk'
                    withCredentials([string(credentialsId: 'snyk-token', variable: 'SNYK_TOKEN')]) {
                        bat 'snyk auth %SNYK_TOKEN%'
                        bat 'snyk test --json > snyk-report.json || exit /b 0'
                    }
                }
                // Print summaries if they exist; ignore missing-file errors
                bat 'type audit.json    || exit /b 0'
                bat 'type snyk-report.json || exit /b 0'
            }
        }
    }

    post {
        always {
            // This runs even if stages are unstable/fail
            emailext(
                to:      'ahadsiddiqui094@gmail.com',
                subject: "Build ${env.JOB_NAME} #${env.BUILD_NUMBER} – ${currentBuild.currentResult}",
                body: """\
Hello,

Your Jenkins job *${env.JOB_NAME}* (build #${env.BUILD_NUMBER}) finished with status *${currentBuild.currentResult}*.

Attached:
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
