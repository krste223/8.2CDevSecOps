pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps {
        checkout([$class: 'GitSCM',
          branches: [[name: '*/main']],
          userRemoteConfigs: [[url: 'https://github.com/krste223/8.2CDevSecOps.git']]
        ])
      }
    }

    stage('Install dependencies') {
      steps {
        sh 'if [ -f package-lock.json ]; then npm ci; else npm install; fi'
      }
    }

    stage('Run tests') {
      steps {
        sh 'npm test || true'
      }
    }

    stage('Coverage') {
      steps {
        sh 'npm run coverage || true'
      }
    }

    stage('Security Audit') {
      steps {
        sh 'npm audit --json > audit.json || true'
        sh 'npm audit || true'
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: 'audit.json, coverage/**/*', allowEmptyArchive: true
    }
  }
}

