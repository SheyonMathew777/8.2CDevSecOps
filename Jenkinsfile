pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps { checkout scm }   // uses the same repo/branch Jenkins pulls
    }
    stage('Install Dependencies') {
      steps { sh 'npm install' }
    }
    stage('Run Tests') {
      steps { sh 'npm test || true' }     // continue even if tests fail
    }
    stage('Generate Coverage Report') {
      steps { sh 'npm run coverage || true' }
    }
    stage('NPM Audit (Security Scan)') {
      steps { sh 'npm audit || true' }
    }
  }
}
