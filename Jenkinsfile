pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Install') {
      steps {
        // clean, reproducible install
        bat 'npm ci'
      }
    }

    stage('Security Audit') {
      steps {
        // wrap the audit so non-zero exit doesn’t fail the build
        catchError(buildResult: 'UNSTABLE', stageResult: 'UNSTABLE') {
          // dump the full JSON report
          bat 'npm audit --json > audit.json'
        }
      }
    }
  }

  post {
    always {
      // send you an email on SUCCESS or UNSTABLE
      emailext(
        to:      'ahadsiddiqui094@gmail.com',
        subject: "${env.JOB_NAME} #${env.BUILD_NUMBER}: ${currentBuild.currentResult}",
        mimeType: 'text/html',
        body: """
          <p>Build <b>${env.JOB_NAME} #${env.BUILD_NUMBER}</b> finished with <b>${currentBuild.currentResult}</b>.</p>
          <p>See attached <code>audit.json</code> for the full <code>npm audit</code> report.</p>
        """,
        attachmentsPattern: 'audit.json'
      )
    }
  }
}
