pipeline {
  agent any

  environment {
    // Where to send your notifications
    EMAIL_RECIPIENT = 'ahadsiddiqui094@gmail.com'
  }

  stages {
    stage('Checkout') {
      steps {
        // Tool: Git (Jenkins Git Plugin)
        // Purpose: grab the latest code
        checkout([$class: 'GitSCM',
          branches: [[name: '*/main']],
          userRemoteConfigs: [[url: 'https://github.com/ahadsiddiqui/8.1CDevSecOps']]
        ])
      }
    }

    stage('Install Dependencies') {
      steps {
        // Tool: npm
        // Purpose: install project libraries
        bat 'npm ci'
      }
    }

    stage('Run Tests') {
      steps {
        // Tool: npm test (e.g. Mocha/Jest)
        // Purpose: run unit tests, capture stdout to a file
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
          bat 'npm test > testlog.txt || exit 0'
        }
      }
    }

    stage('Security Audit') {
      steps {
        // Tool: npm audit
        // Purpose: scan for vulnerable dependencies
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
          bat 'npm audit > auditlog.txt || exit 0'
        }
      }
    }
  }

  post {
    always {
      // 1) Archive the logs so the Email-Ext plugin can pick them up
      archiveArtifacts artifacts: 'testlog.txt,auditlog.txt', allowEmptyArchive: true

      // 2) Send exactly one email per build
      emailext(
        to:               env.EMAIL_RECIPIENT,
        subject:          "${currentBuild.currentResult}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body:             """\
Hello,

Your pipeline **${env.JOB_NAME}** build **#${env.BUILD_NUMBER}** finished with status **${currentBuild.currentResult}**.

• View the full console log here: ${env.BUILD_URL}console  
• Attached are the test and audit logs for your review.

""".stripIndent(),
        // Make sure there are NO spaces around commas
        attachmentsPattern: 'testlog.txt,auditlog.txt',
        // Also include the entire console log
        attachLog:         true
      )
    }
  }
}
