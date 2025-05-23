pipeline {
  agent any

  // grab your SNYK_TOKEN from Jenkins Credentials (ID = "snyk-token")
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
        bat 'npm ci'
      }
    }

    stage('Authenticate Snyk') {
      steps {
        // npx will download/run the local or remote snyk binary
        bat 'npx snyk auth %SNYK_TOKEN%'
      }
    }

    stage('Run Tests') {
      steps {
        // if tests (or snyk test inside them) fail, mark UNSTABLE but keep going
        catchError(buildResult: 'UNSTABLE', stageResult: 'UNSTABLE') {
          bat 'npm test'
        }
      }
    }

    stage('Security Scans') {
      parallel {
        stage('npm audit') {
          steps {
            catchError(buildResult: 'UNSTABLE', stageResult: 'UNSTABLE') {
              bat 'npm audit --json > audit.json'
            }
          }
        }
        stage('snyk test') {
          steps {
            catchError(buildResult: 'UNSTABLE', stageResult: 'UNSTABLE') {
              bat 'npx snyk test --json > snyk-report.json'
            }
          }
        }
      }
      post {
        always {
          // show the JSON in the console so you can eyeball it in Jenkins without downloading
          bat 'type audit.json         || exit /b 0'
          bat 'type snyk-report.json   || exit /b 0'
        }
      }
    }
  }

  post {
    always {
      // send you an email no matter what, with both reports attached
      emailext(
        to:      'ahadsiddiqui094@gmail.com',
        subject: "Jenkins: ${env.JOB_NAME} #${env.BUILD_NUMBER} — ${currentBuild.currentResult}",
        body: """\
Hello Ahad,

Job:    ${env.JOB_NAME}
Build:  #${env.BUILD_NUMBER}
Status: ${currentBuild.currentResult}

Please find attached:
 • npm audit report (audit.json)
 • snyk report (snyk-report.json)
 • full console log

Regards,
Jenkins
""",
        attachLog:          true,
        attachmentsPattern: 'audit.json,snyk-report.json'
      )
    }
  }
}
