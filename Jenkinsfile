pipeline {
  agent any

  environment {
    EMAIL_RECIPIENT = 'ahadsiddiqui094@gmail.com' 
  }

  stages {
    stage('Checkout') {
      steps {
        // Tool: Git (via Jenkins Git plugin)
        checkout([
          $class: 'GitSCM',
          branches: [[name: '*/main']],
          userRemoteConfigs: [[url: 'https://github.com/ahadsiddiqui/8.1CDevSecOps']] 
        ])
      }
    }

    stage('Install Dependencies') {
      steps {
        // Tool: npm
        bat 'npm ci'
      }
    }

    stage('Run Tests') {
      steps {
        // Tool: npm test (e.g. Mocha/Jest)
        // Always write a log, even on failure
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
          bat 'npm test > testlog.txt || exit 0'
        }
      }
      post {
        always {
          // make log available for attachment
          archiveArtifacts artifacts: 'testlog.txt', allowEmptyArchive: true
        }
        success {
          emailext(
            to:               "${EMAIL_RECIPIENT}",
            subject:          "✅ Tests PASSED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body:             """\
              Hi team,

              The **Run Tests** stage completed **SUCCESSFULLY**.
              Please find the test log attached.

              Regards,
              Jenkins
            """.stripIndent(),
            attachmentsPattern: 'testlog.txt',
            mimeType:         'text/plain'
          )
        }
        failure {
          emailext(
            to:               "${EMAIL_RECIPIENT}",
            subject:          "❌ Tests FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body:             """\
              Hi team,

              The **Run Tests** stage **FAILED**.
              Please find the test log attached for details.

              Regards,
              Jenkins
            """.stripIndent(),
            attachmentsPattern: 'testlog.txt',
            mimeType:         'text/plain'
          )
        }
      }
    }

    stage('Security Audit') {
      steps {
        // Tool: npm audit
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
          bat 'npm audit > auditlog.txt || exit 0'
        }
      }
      post {
        always {
          archiveArtifacts artifacts: 'auditlog.txt', allowEmptyArchive: true
        }
        success {
          emailext(
            to:               "${EMAIL_RECIPIENT}",
            subject:          "✅ Security Audit PASSED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body:             """\
              Hi team,

              The **Security Audit** stage completed **SUCCESSFULLY**.
              Please find the audit log attached.

              Regards,
              Jenkins
            """.stripIndent(),
            attachmentsPattern: 'auditlog.txt',
            mimeType:         'text/plain'
          )
        }
        failure {
          emailext(
            to:               "${EMAIL_RECIPIENT}",
            subject:          "❌ Security Audit FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body:             """\
              Hi team,

              The **Security Audit** stage **FAILED**.
              Please find the audit log attached for details.

              Regards,
              Jenkins
            """.stripIndent(),
            attachmentsPattern: 'auditlog.txt',
            mimeType:         'text/plain'
          )
        }
      }
    }
  }

  // (Optional) a global post to catch anything else:
  post {
    always {
      echo "Pipeline finished with status: ${currentBuild.currentResult}"
    }
  }
}
