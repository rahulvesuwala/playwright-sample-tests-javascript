pipeline {
    agent any
    tools {
        nodejs 'NODEJS 24'
    }
    environment {
        TESTDINO_API_KEY = credentials('testdino-api-key')
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
                sh 'npx playwright install --with-deps'
            }
        }
        stage('Run Playwright Tests') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    sh '''
                    mkdir -p playwright-report
                    npx playwright test --reporter=dot,junit=playwright-report/results.xml,json=playwright-report/report.json
                    '''
                }
            }
        }
        stage('Publish Test Results') {
            steps {
                junit 'playwright-report/results.xml'
                archiveArtifacts artifacts: 'playwright-report/**', allowEmptyArchive: true
            }
        }
        stage('Upload TestDino Report') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    dir('playwright-report') {
                        sh '''
                        if [ -f "report.json" ]; then
                            echo "Uploading TestDino report..."
                            curl -X POST \
                            -H "Authorization: Bearer $TESTDINO_API_KEY" \
                            -F "file=@report.json" \
                            https://api.testdino.com/report/upload
                        else
                            echo "⚠️ No report.json found. Skipping TestDino upload."
                        fi
                        '''
                    }
                }
            }
        }
    }
    post {
        always {
            echo "✅ Pipeline finished. Check Jenkins Test Results tab and TestDino dashboard."
        }
    }
}
