pipeline {
  agent any

  options {
    timestamps()
    buildDiscarder(logRotator(numToKeepStr: '10'))
  }

  // (Optional) keep poll trigger if you want auto-runs on commits
  // triggers { pollSCM('H/2 * * * *') }

  environment {
    EMAIL_TO   = 'krste.name@gmail.com'
    EMAIL_FROM = 'krste.name@gmail.com'
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

    stage('Tests') {
      steps {
        // run tests, capture exit code and log to file
        sh '''
          set +e
          npm test | tee test.log
          echo $? > test.exit
          set -e
        '''
      }
      post {
        always {
          script {
            def code = readFile('test.exit').trim()
            def ok   = (code == '0')
            emailext(
              subject: (ok ? "✅ Tests PASSED – ${env.JOB_NAME} #${env.BUILD_NUMBER}"
                         : "❌ Tests FAILED – ${env.JOB_NAME} #${env.BUILD_NUMBER}"),
              body: """Stage: Tests
Status: ${ok ? 'SUCCESS' : 'FAILURE'}
Job: ${env.JOB_NAME} #${env.BUILD_NUMBER}
Console: ${env.BUILD_URL}console
Artifacts: ${env.BUILD_URL}artifact/
""",
              to:    env.EMAIL_TO,
              from:  env.EMAIL_FROM,
              replyTo: env.EMAIL_FROM,
              attachLog: true,
              compressLog: true,
              // attach the stage log file
              attachmentsPattern: 'test.log'
            )
          }
        }
      }
    }

    stage('Security Audit') {
      steps {
        // create both JSON (machine) and text (human) outputs
        sh '''
          set +e
          npm audit --json > audit.json
          echo $? > audit.exit
          npm audit > audit.txt 2>&1 || true
          set -e
        '''
      }
      post {
        always {
          script {
            def code = readFile('audit.exit').trim()
            def ok   = (code == '0')  // npm audit returns non-zero if vulns found
            emailext(
              subject: (ok ? "✅ Security Audit CLEAN – ${env.JOB_NAME} #${env.BUILD_NUMBER}"
                         : "⚠️ Security Audit FOUND VULNS – ${env.JOB_NAME} #${env.BUILD_NUMBER}"),
              body: """Stage: Security Audit
Status: ${ok ? 'NO VULNERABILITIES REPORTED' : 'VULNERABILITIES DETECTED'}
Job: ${env.JOB_NAME} #${env.BUILD_NUMBER}
Console: ${env.BUILD_URL}console
Artifacts (include audit.json & audit.txt): ${env.BUILD_URL}artifact/
""",
              to:    env.EMAIL_TO,
              from:  env.EMAIL_FROM,
              replyTo: env.EMAIL_FROM,
              attachLog: true,
              compressLog: true,
              // attach human-readable audit and keep JSON as artifact
              attachmentsPattern: 'audit.txt'
            )
          }

          // archive logs & JSON so they appear in “Build Artifacts”
          archiveArtifacts artifacts: 'test.log, audit.txt, audit.json', allowEmptyArchive: true
        }
      }
    }
  }

  post {
    always {
      // keep workspace outputs for inspection if needed
      echo "Build finished with status: ${currentBuild.currentResult}"
    }
  }
}
