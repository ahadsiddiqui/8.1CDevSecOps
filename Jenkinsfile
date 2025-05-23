pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                // Pull the Jenkinsfile and code from your GitHub repo
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                // Install Node.js packages
                bat 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                // Run tests but don’t fail the build if they fail
                bat 'npm test || exit /b 0'
            }
        }

        stage('Generate Coverage Report') {
            steps {
                // Generate a coverage report without breaking the build
                bat 'npm run coverage || exit /b 0'
            }
        }

        stage('Security Scan') {
            steps {
                // Audit for known vulnerabilities and continue on error
                bat 'npm audit --json > audit.json || exit /b 0'
                // Display the audit summary in the console
                bat 'type audit.json'
            }
        }
    }

    post {
        always {
            // Send a custom email after every build
            emailext(
                to: 'ahadsiddiqui094@gmail.com',
                subject: "Jenkins Build ${env.JOB_NAME} #${env.BUILD_NUMBER} - ${currentBuild.currentResult}",
                body: """\
Hello,

Your Jenkins job *${env.JOB_NAME}* (build #${env.BUILD_NUMBER}) has finished with status: *${currentBuild.currentResult}*.

– Audit report is attached as **audit.json**  
– Full console log is attached as **log.txt**

Regards,  
Jenkins
""",
                attachLog: true,
                attachmentsPattern: 'audit.json'
            )
        }
    }
}
