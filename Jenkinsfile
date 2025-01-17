pipeline {
    agent any
    environment {
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }
    stages {
        stage('Docker') {
            steps {
                sh 'docker build -t my-playwright .'
            }
        }
        stage('build') {
            agent{
                docker {
                    image 'my-playwright'
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
                            image 'my-playwright'
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
                            image 'my-playwright'
                            reuseNode true
                        }
                    }
                    steps{
                        sh '''
                            serve -s build &
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
        stage('Deploy To Stage'){
            agent{
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }
            environment {
                 NETLIFY_SITE_ID = '6f45bac1-7c6a-42f5-9b78-24baaf245332'
            }
            steps{
                sh '''
                node --version
                npm  --version
               
                netlify --version
                netlify login
                netlify status
                netlify deploy --dir=build 
                '''
            }
            
        }

        stage('Deploy To Prod'){
            agent{
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }
            environment {
                NETLIFY_SITE_ID = 'd09f099f-1d1e-41ca-9eb4-fddaccb998d7'
            }
            steps{
                sh '''
                node --version
                npm  --version
               
                netlify --version
                netlify login
                netlify status
                netlify deploy --dir=build --prod
                '''
            }
            
        }
        stage('E2E Prod'){
            agent{
                docker {
                    image 'my-playwright'
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
