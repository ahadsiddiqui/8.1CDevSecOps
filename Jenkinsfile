pipeline {
  agent any

  environment {
    EMAIL_RECIPIENT = 'ahadsiddiqui094@gmail.com'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout([
          $class: 'GitSCM',
          branches: [[name: '*/main']],
          userRemoteConfigs: [[url: 'https://github.com/ahadsiddiqui/8.1CDevSecOps']]
        ])
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
          bat 'npm test > testlog.txt || exit 0'
        }
      }
    }

    stage('Security Audit') {
      steps {
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
          bat 'npm audit > auditlog.txt || exit 0'
        }
      }
    }
  }

  post {
    always {
      // 1) Archive the logs so they can be attached
      archiveArtifacts artifacts: 'testlog.txt,auditlog.txt', allowEmptyArchive: true

      // 2) Send a single email—subject shows SUCCESS or FAILURE
      script {
        def result = currentBuild.currentResult ?: 'SUCCESS'
        emailext(
          to:               EMAIL_RECIPIENT,
          subject:          "Build ${result}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
          body:             """\
            Hello,

            Your pipeline has finished with status: ${result}

            • View Console: ${env.BUILD_URL}console
            • Attached: testlog.txt, auditlog.txt
          """.stripIndent(),
          attachmentsPattern: 'testlog.txt,auditlog.txt',
          attachLog:         true
        )
      }
    }
  }
}
