pipeline {
  agent any

  stages {
    stage('Checkout')     { steps { checkout scm } }
    stage('Install')      { steps { bat 'npm install' } }

    stage('Run Tests') {
      steps {
        // make sure snyk is installed & authenticated
        bat 'npm install -g snyk'
        withCredentials([string(credentialsId: 'snyk-token', variable: 'SNYK_TOKEN')]) {
          bat 'snyk auth %SNYK_TOKEN%'
        }

        // catch any test failures (including snyk) so build stays SUCCESS
        catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
          bat 'npm test'
        }
      }
    }

    stage('Generate Coverage') {
      steps {
        // if you don’t have a coverage script, you can skip or guard this
        bat 'npm run coverage || exit /b 0'
      }
    }

    stage('Security Scans') {
      steps {
        // npm audit
        catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
          bat 'npm audit --json > audit.json'
        }

        // snyk test again if you like
        catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
          bat 'snyk test --json > snyk-report.json'
        }

        // show results in Console
        bat 'type audit.json         || exit /b 0'
        bat 'type snyk-report.json   || exit /b 0'
      }
    }
  }

  post {
    always {
      emailext(
        to:      'ahadsiddiqui094@gmail.com',
        subject: "Build ${env.JOB_NAME} #${env.BUILD_NUMBER}: ${currentBuild.currentResult}",
        body: """\
Hello,

Job:    ${env.JOB_NAME}
Build:  #${env.BUILD_NUMBER}
Status: ${currentBuild.currentResult}

Attached:
 - audit.json
 - snyk-report.json
 - Full console log

Regards,
Jenkins
""",
        attachLog:          true,
        attachmentsPattern: 'audit.json,snyk-report.json'
      )
    }
  }
}
