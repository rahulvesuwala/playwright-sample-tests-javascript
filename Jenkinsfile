pipeline {
    agent any
    tools {
        nodejs 'NODEJS 24'  // Name of NodeJS installation in Jenkins
    }
    environment {
        TESTDINO_API_KEY = credentials('testdino-api-key') // Your TestDino API key
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
                // Run tests but continue pipeline even if some tests fail
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    sh 'npx playwright test --reporter=dot,junit,json'
                }
            }
        }
        stage('Publish Test Results') {
            steps {
                // Publish JUnit XML reports to Jenkins
                junit 'playwright-report/*.xml'
                // Archive reports as artifacts
                archiveArtifacts artifacts: 'playwright-report/**', allowEmptyArchive: true
            }
        }
        stage('Upload TestDino Report') {
            steps {
                // Ensure this runs even if previous stage failed
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
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
    post {
        always {
            echo "Pipeline finished. Check test results and TestDino report."
        }
    }
}
