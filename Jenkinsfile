pipeline {
  agent any

  environment {
    // Make sure you have a Jenkins “Secret text” credential named “snyk-token”
    SNYK_TOKEN = credentials('snyk-token')
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Install Dependencies') {
      steps {
        // Installs everything, including Snyk as a dev dependency
        bat 'npm ci'
        // Add snyk into your devDependencies so that `npx snyk` works:
        bat 'npm install --no-save snyk'
      }
    }

    stage('Snyk Security Scan') {
      steps {
        script {
          // Authenticate via npx (no need for global PATH hacks)
          bat "npx snyk auth %SNYK_TOKEN%"

          // Run the scan and dump JSON.  We capture exit code so pipeline never dies.
          def code = bat(returnStatus: true,
                         script: 'npx snyk test --json > snyk-report.json')

          if (code != 0) {
            currentBuild.result = 'UNSTABLE'
            echo "⚠️  Snyk found issues (exit ${code}); report in snyk-report.json"
          } else {
            echo "✅  Snyk scan passed with no vulnerabilities."
          }
        }
      }
    }
  }

  post {
    always {
      // Always fire an email, attaching the JSON report
      emailext(
        to: 'ahadsiddiqui094@gmail.com',
        subject: "${env.JOB_NAME} #${env.BUILD_NUMBER}: ${currentBuild.currentResult}",
        mimeType: 'text/html',
        body: """
<html>
  <body>
    <p>Build <b>${env.JOB_NAME} #${env.BUILD_NUMBER}</b> finished with status: <b>${currentBuild.currentResult}</b>.</p>
    <p>Please see the attached <code>snyk-report.json</code> for details.</p>
  </body>
</html>
""",
        attachmentsPattern: 'snyk-report.json'
      )
    }
  }
}
