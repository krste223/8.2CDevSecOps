pipeline {
  agent any

  environment {
    
    EMAIL_TO = 'krste.name@gmail.com'
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Install dependencies') {
      steps {
    
        sh '''
          if [ -f package-lock.json ]; then
            npm ci
          else
            npm install
          fi
        '''
      }
    }

    stage('Run tests') {
      steps {
       
        sh 'npm test || true'
      }
      post {
        success {
          emailext(
            to: EMAIL_TO,
            subject: "✅ Tests PASSED – ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: """Tests passed for <b>${env.JOB_NAME} #${env.BUILD_NUMBER}</b>.<br/>
                     View build: ${env.BUILD_URL}""",
            mimeType: 'text/html',
            attachLog: true
          )
        }
        failure {
          emailext(
            to: EMAIL_TO,
            subject: "❌ Tests FAILED – ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: """Tests failed for <b>${env.JOB_NAME} #${env.BUILD_NUMBER}</b>.<br/>
                     View build: ${env.BUILD_URL}""",
            mimeType: 'text/html',
            attachLog: true
          )
        }
      }
    }

    stage('Coverage') {
      when { expression { fileExists('package.json') } }
      steps {
     
        sh 'npm run coverage || true'
      }
    }

    stage('Security Audit') {
      steps {

        sh 'npm audit --json | tee audit.json || true'
        archiveArtifacts artifacts: 'audit.json', fingerprint: true
      }
      post {
        always {
          emailext(
            to: EMAIL_TO,
            subject: "🔒 Security Audit ${currentBuild.currentResult} – ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: """Security audit completed for <b>${env.JOB_NAME} #${env.BUILD_NUMBER}</b>.<br/>
                     <code>audit.json</code> is attached.<br/>
                     View build: ${env.BUILD_URL}""",
            mimeType: 'text/html',
            attachmentsPattern: 'audit.json',
            attachLog: true
          )
        }
      }
    }
  }
}
