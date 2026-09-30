// ═══════════════════════════════════════════════════════════════
// Jenkinsfile — Master CI/CD Pipeline (Windows)
// Playwright TypeScript Framework — Without Docker
// Naveen Automation Labs
// ═══════════════════════════════════════════════════════════════

pipeline {
    agent any

    tools {
        nodejs 'NodeJS-24'
        maven 'Maven-3.9'
        jdk 'JDK-17'
        allure 'Allure'
    }

    parameters {
        choice(name: 'ENVIRONMENT', choices: ['QA', 'dev', 'stage', 'Prod'], description: 'Select environment')
        choice(name: 'BROWSER', choices: ['chromium', 'firefox', 'webkit'], description: 'Select browser')
        choice(name: 'TEST_SUITE', choices: ['all', 'smoke', 'regression', 'api-smoke'], description: 'Select test suite')
    }

    environment {
        SLACK_CHANNEL = '#general'
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '20'))
        disableConcurrentBuilds()
    }

    stages {

        stage('Build & Unit Tests') {
            steps {
                echo '========================================='
                echo '  Building App + Running Unit Tests'
                echo '========================================='
                dir('dev-app') {
                    git url: 'https://github.com/jglick/simple-maven-project-with-tests.git', branch: 'master'
                    bat 'mvn clean install -Dmaven.test.failure.ignore=true'
                }
            }
            post {
                always {
                    junit 'dev-app\\target\\surefire-reports\\*.xml'
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                echo '========================================='
                echo '  Installing Playwright Dependencies'
                echo '========================================='
                dir('qa-tests') {
                    git url: 'https://github.com/naveenanimation20/OpenCartWebAPIFramework.git', branch: 'main'
                    bat 'npm ci'
                    bat 'npx playwright install --with-deps chromium'
                }
            }
        }

        stage('Deploy to DEV') {
            steps { echo 'Deploying to DEV... done' }
        }

        stage('DEV - Sanity Tests') {
            steps {
                echo 'Running SANITY @smoke on DEV'
                dir('qa-tests') {
                    bat 'if exist allure-results rmdir /s /q allure-results'
                    bat 'if exist reports rmdir /s /q reports'
                    withCredentials([
                        usernamePassword(credentialsId: 'dev-credentials', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD'),
                        string(credentialsId: 'api-token', variable: 'API_TOKEN'),
                        string(credentialsId: 'oauth-client-id', variable: 'OAUTH_CLIENT_ID'),
                        string(credentialsId: 'oauth-client-secret', variable: 'OAUTH_CLIENT_SECRET'),
                        string(credentialsId: 'dev-base-url', variable: 'BASE_URL'),
                        string(credentialsId: 'api-base-url', variable: 'API_BASE_URL')
                    ]) {
                        bat """
                            set ENV=dev
                            set BASE_URL=%BASE_URL%
                            set USERNAME=%USERNAME%
                            set PASSWORD=%PASSWORD%
                            set API_BASE_URL=%API_BASE_URL%
                            set API_TOKEN=%API_TOKEN%
                            set OAUTH_CLIENT_ID=%OAUTH_CLIENT_ID%
                            set OAUTH_CLIENT_SECRET=%OAUTH_CLIENT_SECRET%
                            set GRANT_TYPE=client_credentials
                            npx playwright test --project=chromium --grep @smoke
                        """
                    }
                }
            }
            post {
                always {
                    bat 'if not exist reports-dev\\html mkdir reports-dev\\html'
                    bat 'if not exist reports-dev\\allure mkdir reports-dev\\allure'
                    bat 'xcopy /s /e /y /q qa-tests\\reports\\html-report\\* reports-dev\\html\\ 2>nul || exit /b 0'
                    bat 'allure generate qa-tests\\allure-results --clean -o reports-dev\\allure 2>nul || exit /b 0'
                    publishHTML(target: [reportName: 'DEV Sanity - PW HTML Report', reportDir: 'reports-dev\\html', reportFiles: 'index.html', keepAll: true, alwaysLinkToLastBuild: true])
                    publishHTML(target: [reportName: 'DEV Sanity - Allure Report', reportDir: 'reports-dev\\allure', reportFiles: 'index.html', keepAll: true, alwaysLinkToLastBuild: true])
                }
            }
        }

        stage('Deploy to QA') {
            steps { echo 'Deploying to QA... done' }
        }

        stage('QA - Regression Tests') {
            steps {
                echo 'Running REGRESSION (all tests) on QA'
                dir('qa-tests') {
                    bat 'if exist allure-results rmdir /s /q allure-results'
                    bat 'if exist reports rmdir /s /q reports'
                    withCredentials([
                        usernamePassword(credentialsId: 'qa-credentials', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD'),
                        string(credentialsId: 'api-token', variable: 'API_TOKEN'),
                        string(credentialsId: 'oauth-client-id', variable: 'OAUTH_CLIENT_ID'),
                        string(credentialsId: 'oauth-client-secret', variable: 'OAUTH_CLIENT_SECRET'),
                        string(credentialsId: 'qa-base-url', variable: 'BASE_URL'),
                        string(credentialsId: 'api-base-url', variable: 'API_BASE_URL')
                    ]) {
                        bat """
                            set ENV=qa
                            set BASE_URL=%BASE_URL%
                            set USERNAME=%USERNAME%
                            set PASSWORD=%PASSWORD%
                            set API_BASE_URL=%API_BASE_URL%
                            set API_TOKEN=%API_TOKEN%
                            set OAUTH_CLIENT_ID=%OAUTH_CLIENT_ID%
                            set OAUTH_CLIENT_SECRET=%OAUTH_CLIENT_SECRET%
                            set GRANT_TYPE=client_credentials
                            npx playwright test --project=chromium
                        """
                    }
                }
            }
            post {
                always {
                    bat 'if not exist reports-qa\\html mkdir reports-qa\\html'
                    bat 'if not exist reports-qa\\allure mkdir reports-qa\\allure'
                    bat 'xcopy /s /e /y /q qa-tests\\reports\\html-report\\* reports-qa\\html\\ 2>nul || exit /b 0'
                    bat 'allure generate qa-tests\\allure-results --clean -o reports-qa\\allure 2>nul || exit /b 0'
                    publishHTML(target: [reportName: 'QA Regression - PW HTML Report', reportDir: 'reports-qa\\html', reportFiles: 'index.html', keepAll: true, alwaysLinkToLastBuild: true])
                    publishHTML(target: [reportName: 'QA Regression - Allure Report', reportDir: 'reports-qa\\allure', reportFiles: 'index.html', keepAll: true, alwaysLinkToLastBuild: true])
                }
            }
        }

        stage('Deploy to STAGE') {
            steps { echo 'Deploying to STAGE... done' }
        }

        stage('STAGE - Sanity Tests') {
            steps {
                echo 'Running SANITY @smoke on STAGE'
                dir('qa-tests') {
                    bat 'if exist allure-results rmdir /s /q allure-results'
                    bat 'if exist reports rmdir /s /q reports'
                    withCredentials([
                        usernamePassword(credentialsId: 'stage-credentials', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD'),
                        string(credentialsId: 'api-token', variable: 'API_TOKEN'),
                        string(credentialsId: 'oauth-client-id', variable: 'OAUTH_CLIENT_ID'),
                        string(credentialsId: 'oauth-client-secret', variable: 'OAUTH_CLIENT_SECRET'),
                        string(credentialsId: 'stage-base-url', variable: 'BASE_URL'),
                        string(credentialsId: 'api-base-url', variable: 'API_BASE_URL')
                    ]) {
                        bat """
                            set ENV=stage
                            set BASE_URL=%BASE_URL%
                            set USERNAME=%USERNAME%
                            set PASSWORD=%PASSWORD%
                            set API_BASE_URL=%API_BASE_URL%
                            set API_TOKEN=%API_TOKEN%
                            set OAUTH_CLIENT_ID=%OAUTH_CLIENT_ID%
                            set OAUTH_CLIENT_SECRET=%OAUTH_CLIENT_SECRET%
                            set GRANT_TYPE=client_credentials
                            npx playwright test --project=chromium --grep @smoke
                        """
                    }
                }
            }
            post {
                always {
                    bat 'if not exist reports-stage\\html mkdir reports-stage\\html'
                    bat 'if not exist reports-stage\\allure mkdir reports-stage\\allure'
                    bat 'xcopy /s /e /y /q qa-tests\\reports\\html-report\\* reports-stage\\html\\ 2>nul || exit /b 0'
                    bat 'allure generate qa-tests\\allure-results --clean -o reports-stage\\allure 2>nul || exit /b 0'
                    publishHTML(target: [reportName: 'STAGE Sanity - PW HTML Report', reportDir: 'reports-stage\\html', reportFiles: 'index.html', keepAll: true, alwaysLinkToLastBuild: true])
                    publishHTML(target: [reportName: 'STAGE Sanity - Allure Report', reportDir: 'reports-stage\\allure', reportFiles: 'index.html', keepAll: true, alwaysLinkToLastBuild: true])
                }
            }
        }

        stage('Approval for PROD') {
            steps {
                input message: 'Deploy to PROD?', ok: 'Yes, Deploy!', submitter: 'admin,naveen'
            }
        }

        stage('Deploy to PROD') {
            steps { echo 'Deploying to PROD... done' }
        }

        stage('PROD - Smoke Tests') {
            steps {
                echo 'Running SMOKE @smoke on PROD'
                dir('qa-tests') {
                    bat 'if exist allure-results rmdir /s /q allure-results'
                    bat 'if exist reports rmdir /s /q reports'
                    withCredentials([
                        usernamePassword(credentialsId: 'prod-credentials', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD'),
                        string(credentialsId: 'api-token', variable: 'API_TOKEN'),
                        string(credentialsId: 'oauth-client-id', variable: 'OAUTH_CLIENT_ID'),
                        string(credentialsId: 'oauth-client-secret', variable: 'OAUTH_CLIENT_SECRET'),
                        string(credentialsId: 'prod-base-url', variable: 'BASE_URL'),
                        string(credentialsId: 'api-base-url', variable: 'API_BASE_URL')
                    ]) {
                        bat """
                            set ENV=prod
                            set BASE_URL=%BASE_URL%
                            set USERNAME=%USERNAME%
                            set PASSWORD=%PASSWORD%
                            set API_BASE_URL=%API_BASE_URL%
                            set API_TOKEN=%API_TOKEN%
                            set OAUTH_CLIENT_ID=%OAUTH_CLIENT_ID%
                            set OAUTH_CLIENT_SECRET=%OAUTH_CLIENT_SECRET%
                            set GRANT_TYPE=client_credentials
                            npx playwright test --project=chromium --grep @smoke
                        """
                    }
                }
            }
            post {
                always {
                    bat 'if not exist reports-prod\\html mkdir reports-prod\\html'
                    bat 'if not exist reports-prod\\allure mkdir reports-prod\\allure'
                    bat 'xcopy /s /e /y /q qa-tests\\reports\\html-report\\* reports-prod\\html\\ 2>nul || exit /b 0'
                    bat 'allure generate qa-tests\\allure-results --clean -o reports-prod\\allure 2>nul || exit /b 0'
                    publishHTML(target: [reportName: 'PROD Smoke - PW HTML Report', reportDir: 'reports-prod\\html', reportFiles: 'index.html', keepAll: true, alwaysLinkToLastBuild: true])
                    publishHTML(target: [reportName: 'PROD Smoke - Allure Report', reportDir: 'reports-prod\\allure', reportFiles: 'index.html', keepAll: true, alwaysLinkToLastBuild: true])
                }
            }
        }
    }

    post {
        always {
            script {
                def buildStatus = currentBuild.currentResult
                def statusEmoji = buildStatus == 'SUCCESS' ? '✅' : '❌'
                def statusColor = buildStatus == 'SUCCESS' ? 'good' : 'danger'

                slackSend(channel: env.SLACK_CHANNEL, color: statusColor, message: "🎭 *Playwright CI/CD Pipeline Report*\n\n*Overall: ${statusEmoji} ${buildStatus}*\n*Environment:* `${params.ENVIRONMENT}`\n*Build:* #${env.BUILD_NUMBER}\n*Duration:* ${currentBuild.durationString.replace(' and counting', '')}\n\n📊 <${env.BUILD_URL}|View Reports>\n🔍 <${env.BUILD_URL}console|Console Logs>")

                emailext(to: 'naveenanimation20@gmail.com,training@naveenautomationlabs.com', subject: "🎭 CI/CD — ${statusEmoji} ${buildStatus} — Build #${env.BUILD_NUMBER}", mimeType: 'text/html', body: "<html><body style='font-family:Arial;padding:20px;background:#f5f5f5'><div style='max-width:700px;margin:0 auto;background:white;border-radius:12px;overflow:hidden'><div style='background:linear-gradient(135deg,#1a1a2e,#16213e);color:white;padding:30px;text-align:center'><h1 style='margin:0'>🎭 Playwright CI/CD Dashboard</h1><span style='display:inline-block;padding:6px 16px;border-radius:20px;font-weight:bold;margin-top:12px;background:${buildStatus == 'SUCCESS' ? '#28a745' : '#dc3545'};color:white'>${statusEmoji} ${buildStatus}</span></div><div style='padding:24px'><table style='width:100%;border-collapse:collapse'><tr><td style='padding:10px;color:#666'>Environment</td><td style='padding:10px;font-weight:bold'>${params.ENVIRONMENT}</td></tr><tr><td style='padding:10px;color:#666'>Build</td><td style='padding:10px;font-weight:bold'>#${env.BUILD_NUMBER}</td></tr><tr><td style='padding:10px;color:#666'>Duration</td><td style='padding:10px;font-weight:bold'>${currentBuild.durationString.replace(' and counting', '')}</td></tr></table></div><div style='background:#f8f9fa;padding:20px 24px'><a href='${env.BUILD_URL}' style='display:inline-block;padding:10px 20px;background:#1a1a2e;color:white;text-decoration:none;border-radius:6px;margin:4px'>📁 Open Jenkins Build</a><a href='${env.BUILD_URL}console' style='display:inline-block;padding:10px 20px;background:#6c757d;color:white;text-decoration:none;border-radius:6px;margin:4px'>🔍 Console Logs</a></div></div></body></html>")
            }
        }
        success { echo 'PIPELINE: SUCCESS' }
        failure { echo 'PIPELINE: FAILED' }
    }
}
