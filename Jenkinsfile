pipeline {
  agent any

  environment {
    EMAIL_RECIPIENT = 'ahadsiddiqui094@gmail.com'
  }

  stages {
    // … your existing Checkout / Install / Test / Audit stages …
  }

  post {
    always {
      // debug: show exactly what files we have
      bat '''
        echo === workspace listing ===
        dir /B /S *.txt
        echo =========================
      '''
      
      // now send one email with attachments
      emailext(
        to:               EMAIL_RECIPIENT,
        subject:          "Build ${currentBuild.currentResult}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body:             """\
          Hello,

          Your pipeline has finished with status: ${currentBuild.currentResult}

          • Console log: ${env.BUILD_URL}console
          • Attached: testlog.txt and auditlog.txt
        """.stripIndent(),
        attachmentsPattern: '**/testlog.txt,**/auditlog.txt',
        attachLog:         true
      )
    }
  }
}
