pipeline {
  agent any

  environment {
    EMAIL_RECIPIENT = 'ahadsiddiqui094@gmail.com'
  }

  stages {
    stage('Generate Hello File') {
      steps {
        // On Windows:
        bat 'echo Hello from Jenkins! > hello.txt'
        // If you ever run on Linux/Unix agents, use:
        // sh  'echo Hello from Jenkins! > hello.txt'
      }
    }
  }

  post {
    always {
      // 1) Archive hello.txt so Email-Ext can attach it
      archiveArtifacts artifacts: 'hello.txt', allowEmptyArchive: false

      // 2) Send a single test email with that file attached
      emailext(
        to:               "${EMAIL_RECIPIENT}",
        subject:          "🔔 Email-Ext Test: hello.txt attached",
        body:             """\
Hello!

This is a test of the Email-Extension plugin.  You should find `hello.txt` attached.

— Jenkins
""".stripIndent(),
        attachmentsPattern: 'hello.txt',
        attachLog:         false
      )
    }
  }
}
