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
      // make sure our logs survive until the email step
      archiveArtifacts artifacts: 'testlog.txt,auditlog.txt', fingerprint: false

      script {
        // Build a subject reflecting the overall status
        def status = currentBuild.currentResult == 'SUCCESS' ? 'SUCCESSFUL' : 'FAILED'
        emailext(
          to:      EMAIL_RECIPIENT,
          subject: "Pipeline ${status}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
          body: """
            Hi,

            The pipeline has finished with status: ${status}.

            You can view the full console log here: ${env.BUILD_URL}console

            Attached are the per-stage logs (testlog.txt & auditlog.txt).

            Cheers,
            Jenkins
          """,
          // attach just those two files
          attachmentsPattern: 'testlog.txt,auditlog.txt',
          // plus the entire console log
          attachLog: true
        )
      }
    }
  }
}
