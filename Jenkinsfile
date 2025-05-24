pipeline {
  agent any

  environment {
    EMAIL_RECIPIENT = 'ahadsiddiqui094@gmail.com'
  }

  stages {
    stage('Checkout') {
      steps {
        // Tool: Git (via Jenkins Git plugin)
        checkout([$class: 'GitSCM',
          branches: [[name: '*/main']],
          userRemoteConfigs: [[url: 'https://github.com/ahadsiddiqui/8.1CDevSecOps']]
        ])
      }
    }

    stage('Install Dependencies') {
      steps {
        // Tool: npm (Node Package Manager)
        bat 'npm ci'
      }
    }

    stage('Run Tests') {
      steps {
        // Tool: npm test
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
          bat 'npm test > testlog.txt || exit 0'
        }
      }
    }

    stage('Security Audit') {
      steps {
        // Tool: npm audit
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
          bat 'npm audit > auditlog.txt || exit 0'
        }
      }
    }
  }

  post {
    always {
      // DEBUG: show where the logs actually are
      bat '''
        echo ==== WORKSPACE ====
        echo %WORKSPACE%
        dir /B /S "%WORKSPACE%\\*.txt"
        echo ===================
      '''

      // Send one email regardless of pass/fail, but subject reflects status
      emailext(
        to:               "${EMAIL_RECIPIENT}",
        subject:          "Build ${currentBuild.currentResult}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body:             """\
          Hello,

          The pipeline has finished with status: ${currentBuild.currentResult}

          • View console output: ${env.BUILD_URL}console
          • Attached: testlog.txt, auditlog.txt

          Cheers,
          Jenkins
        """.stripIndent(),
        attachmentsPattern: '**/testlog.txt,**/auditlog.txt',
        // set attachLog: true if you also want the full Jenkins console log
        attachLog:         false
      )
    }
  }
}
