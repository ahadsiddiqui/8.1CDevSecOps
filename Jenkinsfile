pipeline {
    agent any
    
    environment {
        EMAIL_RECIPIENT = 'ahadsiddiqui094@gmail.com'
    }
    
    stages {
        stage('Checkout') {
            steps {
                // Replace with your Git checkout step
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], userRemoteConfigs: [[url: 'https://github.com/ahadsiddiqui/8.1CDevSecOps']]])
            }
        }
        
        stage('Install Dependencies') {
            steps {
                bat 'npm ci'
            }
        }
        
        stage('Run Tests') {
            steps {
                bat 'npm test > testlog.txt || exit 0'
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
        success {
            emailext subject: "Pipeline Successful: ${currentBuild.fullDisplayName}",
                     body: "Pipeline executed successfully.",
                     to: "${EMAIL_RECIPIENT}",
                     attachLog: true
        }
        failure {
            emailext subject: "Pipeline Failed: ${currentBuild.fullDisplayName}",
                     body: "There was a failure in the pipeline execution.",
                     to: "${EMAIL_RECIPIENT}",
                     attachLog: true
        }
    }
}
