pipeline {
  agent any

  environment {
    // pulls the token from Jenkins credentials
    SNYK_TOKEN = credentials('snyk-token')
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    // … your other stages …

    stage('Snyk Scan') {
      steps {
        // authenticate then run the test
        bat 'snyk auth %SNYK_TOKEN%'
        bat 'snyk test --json > snyk-report.json || exit /b 0'
        bat 'type snyk-report.json'
      }
    }
  }

  post {
    always {
      // email with both audit.json and snyk-report.json
      emailext(
        to: 'ahadsiddiqui094@gmail.com',
        subject: "Build ${env.JOB_NAME} #${env.BUILD_NUMBER} – ${currentBuild.currentResult}",
        body: "See attached security reports.",
        attachmentsPattern: 'audit.json,snyk-report.json',
        attachLog: true
      )
    }
  }
}
