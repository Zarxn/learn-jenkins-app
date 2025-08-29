pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'bc45247e-e6c8-40a1-9575-12b961444c3f'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }

    stages {
        
        stage('Build') {
            agent {
                docker{
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
        
        stage('Run Tests') {
            parallel {
                stage('Test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                    node --version
                    npm --version
                    find build/ -type f -name index.html
                    npm run test
                    '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                stage('E2E') {
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
                    sleep 10
                    npx playwright test --reporter=html
                    '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }
        
        stage('Deploy Staging') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = 'STAGING_URL_TO_BE_SET'
            }
            steps {
                sh '''
                npm install netlify-cli@20.1.1 node-jq
                node_modules/.bin/netlify --version
                echo "Deploying to staging. Side ID: $NETLIFY_SITE_ID"
                node_modules/.bin/netlify status
                node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                CI_ENVIRONMENT_URL=$( node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json)
                npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        // stage('Approval') {
        //     steps {
        //         timeout(time: 1, unit: 'MINUTES') {
        //             input message: 'Ready to deploy to prod?', ok: 'Yes, I am sure'
        //         }
        //     }
        // }

        stage('Deploy Prod') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    environment {
                        CI_ENVIRONMENT_URL = 'https://curious-marzipan-0c9d2b.netlify.app'
                    }
                    steps {
                        sh '''
                        node --version
                        npm install netlify-cli@20.1.1
                        node_modules/.bin/netlify --version
                        echo "Deploying to production. Side ID: $NETLIFY_SITE_ID"
                        node_modules/.bin/netlify status
                        node_modules/.bin/netlify deploy --dir=build --prod
                        npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
        }
    }
}
