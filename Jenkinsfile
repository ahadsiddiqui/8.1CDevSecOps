pipeline {
  agent any
  environment {
    EMAIL_RECIPIENT = 'ahadsiddiqui094@gmail.com'
  }
  stages {
    stage('Hello') {
      steps {
        writeFile file: 'hello.txt', text: 'Hello world!'
      }
    }
  }
  post {
    always {
      archiveArtifacts artifacts: 'hello.txt'
      emailext(
        to:             EMAIL_RECIPIENT,
        subject:        "Test Email with Attachment",
        body:           "See attached hello.txt",
        attachmentsPattern: 'hello.txt'
      )
    }
  }
}
