pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'd03556c3-1d07-4da5-bede-9885fe75235e'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
      /*  stage('Debug') {
            steps {
                sh '''
                    whoami
                    hostname
                    0which docker || true
                    docker --version || true
                '''
            }
        } */ 

        stage('Build') {
            agent {
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
        stage('Test') {
            parallel {
                stage('Unit test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
        
                    steps {
                        sh ''' 
                           test -f build/index.html
                           npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }
            
                stage('E2E Test') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
        
                    steps {
                        sh ''' 
                           npm install serve
                           node_modules/.bin/serve -s build &
                           sleep 20
                           npx playwright test  --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }
        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }
        
        stage('Prod E2E') {
           agent {
               docker {
                   image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                   reuseNode true
               }
           }

            environment {
                CI_ENVIRONMENT_URL = 'https://effulgent-tarsier-c808ea.netlify.app'
           }

        
           steps {
               sh ''' 
                  npx playwright test  --reporter=html
               '''
           }
           post {
               always {
                   publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E', reportTitles: '', useWrapperFileDirectly: true])
               }
           }
       }
    }
}