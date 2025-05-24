pipeline {
  agent any

  environment {
    EMAIL_RECIPIENT = 'ahadsiddiqui094@gmail.com'
  }

  stages {
    stage('Checkout') {
      steps {
        // Tool: Git
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
        // Tool: npm test
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
          bat 'npm test > testlog.txt || exit 0'
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
    }
  }

  post {
    always {
      // 1) archive the two log files so Email-Ext can actually find them
      archiveArtifacts artifacts: 'testlog.txt,auditlog.txt', allowEmptyArchive: true

      // 2) send exactly one email per build
      script {
        // determine overall status
        def result = currentBuild.currentResult ?: 'SUCCESS'
        emailext(
          to:               EMAIL_RECIPIENT,
          subject:          "Build ${result}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
          body:             """
            Hello,

            Your pipeline has finished with status: ${result}

            • Console log: ${env.BUILD_URL}console
            • Attached: testlog.txt and auditlog.txt
          """.stripIndent(),
          // note: no spaces around the commas
          attachmentsPattern: 'testlog.txt,auditlog.txt',
          // also attach the full console log
          attachLog:         true
        )
      }
    }
  }
}
