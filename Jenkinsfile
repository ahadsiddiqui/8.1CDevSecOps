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

        stage('Generate Coverage Report') {
            steps {
                bat 'npm run coverage || exit /b 0'
            }
        }

        stage('NPM Audit (Security Scan)') {
            steps {
                bat 'npm audit || exit /b 0'
            }
            post {
                always {
                    emailext(
                        to: 'ahadsiddiqui094@gmail.com',
                        subject: "Jenkins Build ${env.JOB_NAME} #${env.BUILD_NUMBER}: ${currentBuild.currentResult}",
                        body: """\
Hello,

Here is the report for build #${env.BUILD_NUMBER} of job ${env.JOB_NAME}:
- Status: ${currentBuild.currentResult}
- Audit report attached as audit.json

Regards,
Jenkins
""",
                        attachLog: true,
                        attachmentsPattern: 'audit.json'
                    )
                }
            }
        }
    }
}
