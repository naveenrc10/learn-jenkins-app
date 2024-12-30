pipeline {
    agent any
    environment {
        NETLIFY_SITE_ID = 'd09f099f-1d1e-41ca-9eb4-fddaccb998d7'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }
    stages {
        stage('build') {
            agent{
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
        stage('Tests') {
            parallel {
                stage('Unit test'){
                    agent{
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps{
                        sh '''
                        test -f build/index.html
                        npm test
                        '''
                    }
                    post {
                        always{
                            junit 'jest-results/junit.xml'
                        }
                    }
            
                }
                stage('E2E Local'){
                    agent{
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps{
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test  --reporter=html
                        '''
                    }
                    post {
                        always{
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local', reportTitles: '', useWrapperFileDirectly: true])
                        }
                        
                    }
                }
            }
        }
        stage('Deploy To Prod'){
            agent{
                docker {
                    image 'node:20-alpine'
                    reuseNode true
                }
            }
            steps{
                sh '''
                node --version
                npm --version
                npm install netlify-cli
                node_modules/.bin/netlify --version
                node_modules/.bin/netlify login
                node_modules/.bin/netlify status
                node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
            
        }
        stage('E2E Prod'){
            agent{
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            environment{
                CI_ENVIRONMENT_URL = "https://euphonious-sawine-60b818.netlify.app"
            }
            steps{
                sh '''
                   
                    npx playwright test  --reporter=html
                '''
            }
            post {
                always{
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Prod', reportTitles: '', useWrapperFileDirectly: true])
                }
                
            }
        }
    }
   
}
