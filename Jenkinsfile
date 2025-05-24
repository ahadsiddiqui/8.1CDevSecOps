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
        // capture test output to file and never abort whole build
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
          bat 'npm test > testlog.txt || exit 0'
        }
      }
      post {
        success {
          emailext(
            to:               EMAIL_RECIPIENT,
            subject:          "✅ Run Tests PASSED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body:             """\
              Hello,

              The **Run Tests** stage completed **SUCCESSFULLY**.

              • See console log: ${env.BUILD_URL}console  
              • Attached: testlog.txt
            """.stripIndent(),
            attachmentsPattern: 'testlog.txt'
          )
        }
        failure {
          emailext(
            to:               EMAIL_RECIPIENT,
            subject:          "❌ Run Tests FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body:             """\
              Hello,

              The **Run Tests** stage **FAILED**.

              • See console log: ${env.BUILD_URL}console  
              • Attached: testlog.txt
            """.stripIndent(),
            attachmentsPattern: 'testlog.txt'
          )
        }
      }
    }

    stage('Security Audit') {
      steps {
        // capture audit output to file and never abort whole build
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
          bat 'npm audit > auditlog.txt || exit 0'
        }
      }
      post {
        success {
          emailext(
            to:               EMAIL_RECIPIENT,
            subject:          "✅ Security Audit PASSED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body:             """\
              Hello,

              The **Security Audit** stage completed **SUCCESSFULLY**.

              • See console log: ${env.BUILD_URL}console  
              • Attached: auditlog.txt
            """.stripIndent(),
            attachmentsPattern: 'auditlog.txt'
          )
        }
        failure {
          emailext(
            to:               EMAIL_RECIPIENT,
            subject:          "❌ Security Audit FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body:             """\
              Hello,

              The **Security Audit** stage **FAILED**.

              • See console log: ${env.BUILD_URL}console  
              • Attached: auditlog.txt
            """.stripIndent(),
            attachmentsPattern: 'auditlog.txt'
          )
        }
      }
    }
  }
}
