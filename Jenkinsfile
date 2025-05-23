pipeline {
  agent any

  environment {
    EMAIL_RECIPIENT = 'ahadsiddiqui094@gmail.com'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Install Dependencies') {
      steps {
        bat 'npm ci'
      }
    }

    stage('Run Tests') {
      steps {
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
          bat 'npm test > testlog.txt'
        }
      }
    }

    stage('Security Audit') {
      steps {
        catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
          bat 'npm audit --json > auditlog.txt'
        }
      }
    }
  }

  post {
    always {
      script {
        // Read both logs and decide status
        def t = fileExists('testlog.txt') ? readFile('testlog.txt') : ''
        def a = fileExists('auditlog.txt') ? readFile('auditlog.txt') : ''
        env.EMAIL_STATUS = (t.contains('ERR') || a.contains('"vulnerabilities":') && a.contains('"total":0')==false)
                            ? 'FAILURE' : 'SUCCESS'
      }

      emailext(
        subject: "Jenkins Build ${EMAIL_STATUS}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body: """Hi Ahad,

The Jenkins build for job '${env.JOB_NAME}' (#${env.BUILD_NUMBER}) has finished with status: ${EMAIL_STATUS}.

You can view the full console output at:
${env.BUILD_URL}console

Regards,
Jenkins""",
        to: "${EMAIL_RECIPIENT}",
        mimeType: 'text/plain',
        attachmentsPattern: '**/testlog.txt, **/auditlog.txt'
      )
    }
  }
}
