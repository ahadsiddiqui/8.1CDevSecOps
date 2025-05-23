pipeline {
  agent any

  tools {
    // if you have the NodeJS plugin installed in Jenkins
    nodejs 'NodeJS 16'
  }

  environment {
    RECIPIENT = 'ahadsiddiqui094@gmail.com'
  }

  stages {
    stage('Checkout') {
      steps {
        // Git checkout
        checkout scm
      }
    }

    stage('Install') {
      steps {
        // clean install
        bat 'npm ci'
      }
    }

    stage('Lint') {
      steps {
        // ESLint (assumes you have a script lint in package.json)
        bat 'npm run lint'
      }
    }

    stage('Test') {
      steps {
        // run tests, tee output into test.log, and mark UNSTABLE instead of FAILED
        catchError(buildResult: 'UNSTABLE', stageResult: 'UNSTABLE') {
          bat 'npm test 2>&1 | tee test.log'
        }
      }
      post {
        always {
          // Email after Test, attach test.log
          emailext(
            to:               "${env.RECIPIENT}",
            subject:          "[${env.JOB_NAME} #${env.BUILD_NUMBER}] Test Stage ${currentBuild.currentResult}",
            mimeType:         'text/html',
            body:             """<p>The <b>Test</b> stage completed with status: <b>${currentBuild.currentResult}</b>.</p>
                                <p>Please see attached <code>test.log</code> for details.</p>""",
            attachmentsPattern: 'test.log'
          )
        }
      }
    }

    stage('Security Scan') {
      steps {
        // run npm audit, dump JSON, but never fail the build
        bat 'npm audit --json > audit.json || exit /b 0'
      }
      post {
        always {
          // Email after Security Scan, attach audit.json
          emailext(
            to:               "${env.RECIPIENT}",
            subject:          "[${env.JOB_NAME} #${env.BUILD_NUMBER}] Security Scan completed",
            mimeType:         'text/html',
            body:             """<p>The <b>Security Scan</b> stage completed with status: <b>${currentBuild.currentResult}</b>.</p>
                                <p>Please see attached <code>audit.json</code> for the full report.</p>""",
            attachmentsPattern: 'audit.json'
          )
        }
      }
    }

    stage('Deploy') {
      steps {
        // your deploy steps here
        echo 'Deploying application…'
      }
    }
  }

  post {
    always {
      echo "Pipeline finished with overall status: ${currentBuild.currentResult}"
    }
  }
}
