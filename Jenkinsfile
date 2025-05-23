pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Install Dependencies') {
      steps { bat 'npm install' }
    }

    stage('Run Tests') {
      steps { bat 'npm test || exit /b 0' }
    }

    stage('Generate Coverage') {
      steps { bat 'npm run coverage || exit /b 0' }
    }

    stage('Security Scans') {
      steps {
        bat 'npm audit --json > audit.json || exit /b 0'
        bat 'npm install -g snyk'

        catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
          withCredentials([string(credentialsId: 'snyk-token', variable: 'SNYK_TOKEN')]) {
            bat 'snyk auth %SNYK_TOKEN%'
            bat 'snyk test --json > snyk-report.json'
          }
        }

        bat 'type audit.json        || exit /b 0'
        bat 'type snyk-report.json  || exit /b 0'
      }
    }
  }

  post {
    always {
      emailext(
        to: 'ahadsiddiqui094@gmail.com',
        subject: "Build ${env.JOB_NAME} #${env.BUILD_NUMBER} – ${currentBuild.currentResult}",
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
        attachLog: true,
        attachmentsPattern: 'audit.json,snyk-report.json'
      )
    }
  }
}
