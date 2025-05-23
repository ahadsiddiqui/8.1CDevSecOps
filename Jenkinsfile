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
        // Clean, fast, reproducible install
        bat 'npm ci'
      }
    }

    stage('Security Audit') {
      steps {
        script {
          // 1) dump the full audit report
          bat 'npm audit --json > audit.json'

          // 2) run npm audit (text) and capture exit code
          def auditCode = bat(returnStatus: true, script: 'npm audit')

          if (auditCode != 0) {
            currentBuild.result = 'UNSTABLE'
            echo "⚠️  Vulnerabilities found (exit code ${auditCode}). See audit.json"
          } else {
            echo "✅  No vulnerabilities found."
          }
        }
      }
    }
  }

  post {
    always {
      emailext(
        to:      'ahadsiddiqui094@gmail.com',
        subject: "${env.JOB_NAME} #${env.BUILD_NUMBER}: ${currentBuild.currentResult}",
        mimeType: 'text/html',
        body: """
          <p>Build <b>${env.JOB_NAME} #${env.BUILD_NUMBER}</b> finished with <b>${currentBuild.currentResult}</b>.</p>
          <p>The full <code>npm audit</code> report is attached.</p>
        """,
        attachmentsPattern: 'audit.json'
      )
    }
  }
}
