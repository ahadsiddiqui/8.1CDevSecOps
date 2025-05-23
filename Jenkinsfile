pipeline {
  agent any

  // Inject your Snyk Auth Token from Jenkins → Credentials → System → Global credentials
  environment {
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
        // On Windows, use `npm ci` for a clean install
        bat 'npm ci'
      }
    }

    stage('Install Snyk CLI') {
      steps {
        // Globally install the Snyk CLI so that `snyk` is on PATH
        bat 'npm install -g snyk'
      }
    }

    stage('Snyk Security Scan') {
      steps {
        script {
          // Authenticate the CLI
          bat "snyk auth %SNYK_TOKEN%"

          // Run `snyk test`, capture its exit code, and write JSON report
          def exitCode = bat(returnStatus: true,
                             script: 'snyk test --json > snyk-report.json')

          // If Snyk found vulnerabilities, mark the build UNSTABLE (but not FAILED)
          if (exitCode != 0) {
            currentBuild.result = 'UNSTABLE'
            echo "⚠️  Snyk found vulnerabilities (exit ${exitCode}). See snyk-report.json"
          } else {
            echo "✅  Snyk scan passed cleanly."
          }
        }
      }
    }
  }

  post {
    // Always email you, regardless of SUCCESS or UNSTABLE
    always {
      emailext(
        to:      'ahadsiddiqui094@gmail.com',
        subject: "${env.JOB_NAME} #${env.BUILD_NUMBER}: ${currentBuild.currentResult}",
        body: """\
<html>
  <body>
    <p>Build <b>${env.JOB_NAME} #${env.BUILD_NUMBER}</b> finished with status: <b>${currentBuild.currentResult}</b>.</p>
    <p>The Snyk JSON report is attached for details.</p>
  </body>
</html>
""",
        mimeType:        'text/html',
        attachmentsPattern: 'snyk-report.json'
      )
    }
  }
}
