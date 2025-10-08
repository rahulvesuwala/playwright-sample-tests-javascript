pipeline {
    agent any
    environment {
        TESTDINO_API_KEY = credentials('testdino-api-key') // Add your TestDino API key in Jenkins
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/rahulvesuwala/playwright-sample-tests-javascript.git', credentialsId: 'github-creds'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'npm ci'
            }
        }
        stage('Run Playwright Tests') {
            steps {
                sh 'npx playwright test --reporter=list,json'
            }
        }
        stage('Publish Test Results') {
            steps {
                junit '**/playwright-report/*.xml'
                archiveArtifacts artifacts: '**/playwright-report/**', allowEmptyArchive: true
            }
        }
        stage('Upload TestDino Report') {
            steps {
                sh '''
                curl -X POST \
                -H "Authorization: Bearer $TESTDINO_API_KEY" \
                -F "file=@playwright-report/report.json" \
                https://api.testdino.com/report/upload
                '''
            }
        }
    }
}
