pipeline {
    agent any

    tools {
        nodejs 'NODEJS 24'  // NodeJS installation in Jenkins
    }

    environment {
        TESTDINO_API_KEY = credentials('testdino-api-key') // Your TestDino API key
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/rahulvesuwala/playwright-sample-tests-javascript.git',
                    credentialsId: 'github-creds'
            }
        }

        stage('Check NodeJS') {
            steps {
                sh 'node -v'
                sh 'npm -v'
                sh 'npx -v'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Install Playwright Browsers') {
            steps {
                sh 'npx playwright install'
            }
        }

        stage('Run Playwright Tests') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    // Run tests with XML and JSON reports in playwright-report folder
                    sh 'npx playwright test'
                }
            }
        }

        stage('Merge Reports (Optional)') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    sh 'npx playwright merge-reports --config=playwright.config.js ./playwright-report'
                }
            }
        }

        stage('Publish Test Results') {
            steps {
                script {
                    if (fileExists('playwright-report/*.xml')) {
                        junit 'playwright-report/*.xml'
                        archiveArtifacts artifacts: 'playwright-report/**', allowEmptyArchive: true
                    } else {
                        echo "No XML reports found. Skipping publish."
                    }
                }
            }
        }

        stage('Upload TestDino Report') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    sh '''
                    NODE_ENV="staging" npx --yes tdpw ./playwright-report/ --token="$TESTDINO_API_KEY"
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
