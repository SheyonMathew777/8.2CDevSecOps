pipeline {
  agent any

  environment {
    // Where to send notifications
    NOTIFY_EMAIL = 'sheyonmathew777@gmail.com'
  }

  options {
    timestamps()
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Install Dependencies') {
      steps {
        // Use npm ci if lockfile is present, fallback to npm install
        sh 'if [ -f package-lock.json ]; then npm ci; else npm install; fi'
      }
    }

    stage('Run Tests') {
      steps {
        // Mark the stage as FAILURE if tests fail, but keep the pipeline going
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
          sh 'npm test'
        }
        // Save anything useful for email attachments / artifacts
        archiveArtifacts artifacts: 'coverage/**/*, **/junit*.xml, **/test-results/**/*', allowEmptyArchive: true
      }
      post {
        success {
          emailext(
            to: env.NOTIFY_EMAIL,
            subject: "[${env.JOB_NAME}] ✔ Tests passed — #${env.BUILD_NUMBER}",
            body: """Build URL: ${env.BUILD_URL}
Stage: Run Tests
Status: SUCCESS""",
            attachLog: true,
            attachmentsPattern: "coverage/**/*, **/junit*.xml, **/test-results/**/*"
          )
        }
        failure {
          emailext(
            to: env.NOTIFY_EMAIL,
            subject: "[${env.JOB_NAME}] ✖ Tests FAILED — #${env.BUILD_NUMBER}",
            body: """Build URL: ${env.BUILD_URL}
Stage: Run Tests
Status: FAILURE""",
            attachLog: true,
            attachmentsPattern: "coverage/**/*, **/junit*.xml, **/test-results/**/*"
          )
        }
      }
    }

    stage('Generate Coverage Report') {
      steps {
        // Don't fail the whole build if coverage task isn't defined
        sh 'npm run coverage || true'
        archiveArtifacts artifacts: 'coverage/**/*', allowEmptyArchive: true
      }
    }

    stage('NPM Audit (Security Scan)') {
      steps {
        // Produce a JSON report; mark the stage failure if issues cause non-zero exit
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
          sh 'npm audit --json > npm-audit.json'
        }
        archiveArtifacts artifacts: 'npm-audit.json', allowEmptyArchive: true
      }
      post {
        success {
          emailext(
            to: env.NOTIFY_EMAIL,
            subject: "[${env.JOB_NAME}] ✔ Security scan clean — #${env.BUILD_NUMBER}",
            body: """Build URL: ${env.BUILD_URL}
Stage: NPM Audit (Security Scan)
Status: SUCCESS""",
            attachLog: true,
            attachmentsPattern: "npm-audit.json"
          )
        }
        failure {
          emailext(
            to: env.NOTIFY_EMAIL,
            subject: "[${env.JOB_NAME}] ⚠ Security issues found — #${env.BUILD_NUMBER}",
            body: """Build URL: ${env.BUILD_URL}
Stage: NPM Audit (Security Scan)
Status: FAILURE""",
            attachLog: true,
            attachmentsPattern: "npm-audit.json"
          )
        }
      }
    }
  }
}
