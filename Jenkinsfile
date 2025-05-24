pipeline {
  agent any

  environment {
    EMAIL_RECIPIENT = 'ahadsiddiqui094@gmail.com'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout([$class: 'GitSCM',
          branches: [[name: '*/main']],
          userRemoteConfigs: [[url: 'https://github.com/ahadsiddiqui/8.1CDevSecOps']]
        ])
      }
    }

    stage('Install Dependencies') {
      steps {
        bat 'npm ci'
      }
    }

    stage('Sanity Check') {
      steps {
        // Tool: npm (custom sanity script)
        // Purpose: quick smoke tests or basic health checks
        bat 'npm run sanity'
      }
    }

    stage('Run Tests') {
      steps {
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
          bat 'npm test > testlog.txt || exit 0'
        }
      }
    }

    stage('Security Audit') {
      steps {
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
          bat 'npm audit > auditlog.txt || exit 0'
        }
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: 'testlog.txt,auditlog.txt', allowEmptyArchive: true
      emailext(
        to:               env.EMAIL_RECIPIENT,
        subject:          "${currentBuild.currentResult}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body:             """\
Hello,

Your pipeline **${env.JOB_NAME}** build **#${env.BUILD_NUMBER}** finished with status **${currentBuild.currentResult}**.

• View the full console log here: ${env.BUILD_URL}console  
• Attached are the test and audit logs.

""".stripIndent(),
        attachmentsPattern: 'testlog.txt,auditlog.txt',
        attachLog:         true
      )
    }
  }
}
