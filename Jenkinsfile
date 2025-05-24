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
        // capture stdout+stderr to testlog.txt, but never abort the whole job
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
          bat 'npm test > testlog.txt 2>&1'
        }
      }
      post {
        always {
          script {
            def status = currentBuild.currentResult
            emailext(
              to:               EMAIL_RECIPIENT,
              subject:          (status == 'SUCCESS' ? "✅ Tests PASSED" : "❌ Tests FAILED") + ": ${env.JOB_NAME} #${env.BUILD_NUMBER}",
              body:             """\
                Hello,

                The **Run Tests** stage has finished with status: ${status}

                • View full console: ${env.BUILD_URL}console
                • Attached: testlog.txt
              """.stripIndent(),
              attachmentsPattern: 'testlog.txt'
            )
          }
        }
      }
    }

    stage('Security Audit') {
      steps {
        // capture audit output to auditlog.txt, but never abort the whole job
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
          bat 'npm audit > auditlog.txt 2>&1'
        }
      }
      post {
        always {
          script {
            def status = currentBuild.currentResult
            emailext(
              to:               EMAIL_RECIPIENT,
              subject:          (status == 'SUCCESS' ? "✅ Security Audit PASSED" : "❌ Security Audit FAILED") + ": ${env.JOB_NAME} #${env.BUILD_NUMBER}",
              body:             """\
                Hello,

                The **Security Audit** stage has finished with status: ${status}

                • View full console: ${env.BUILD_URL}console
                • Attached: auditlog.txt
              """.stripIndent(),
              attachmentsPattern: 'auditlog.txt'
            )
          }
        }
      }
    }
  }
}
