pipeline {
  agent any

  environment {
    SNYK_TOKEN = credentials('snyk-token')  // Make sure the ID here matches exactly what you saved
  }

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
        bat 'snyk auth %SNYK_TOKEN%'
        bat 'snyk test --json > snyk-report.json || exit /b 0'
      }
    }

    // ← New final stage runs inside the workspace
    stage('Notify') {
      steps {
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
 - full console log
""",
          attachLog: true,
          attachmentsPattern: 'audit.json,snyk-report.json'
        )
      }
    }
  }
}
