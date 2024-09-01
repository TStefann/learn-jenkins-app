pipeline {
    agent any

    environment{
        NETLIFY_SITE_ID= 'f2be4442-6706-4c54-9266-de41ce6d6112'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = '1.0.$BUILD_ID'
    }

    stages {
        stage('Run Tests') {
            parallel {
                stage('Unit Test') {
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
                
                stage('e2e test') {
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
                            publishHTML([
                                allowMissing: false,
                                alwaysLinkToLastBuild: false,
                                keepAll: false,
                                reportDir: 'playwright-report',
                                reportFiles: 'index.html',
                                reportName: 'Playwright HTML Report',
                                reportTitles: '',
                                useWrapperFileDirectly: true
                            ])
                        }
                    }
                }

            }
        }

                }
        stage('Deploy Stage') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            steps {
                sh ''' 
                npm install netlify-cli node-jq
                node_modules/.bin/netlify --version
                echo "Deployng to staging. Site ID: $NETLIFY_SITE_ID"
                node_modules/.bin/netlify status 
                node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                CI_ENVIRONMENT_URL=$(node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json)
                npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: false,
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'Staging E2E',
                        reportTitles: '',
                        useWrapperFileDirectly: true
                    ])
                }
            }
        }


                // stage('Approval'){
                //     steps{
                //         timeout(time: 15, unit: 'MINUTES'){
                //             input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure' 
                //         }
                        
                //     }
                // }
            
        stage('Deploy prod') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

        environment{
            CI_ENVIRONMENT_URL= 'https://66d46f75fa719039cf628266--storied-blancmange-03b508.netlify.app'
        }

            steps {
                sh ''' 
                node --version
                npm install netlify-cli
                node_modules/.bin/netlify --version
                echo "Deployng to production. Site ID: $NETLIFY_SITE_ID"
                node_modules/.bin/netlify status
                node_modules/.bin/netlify deploy --dir=build --prod
                npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: false,
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'Production E2E',
                        reportTitles: '',
                        useWrapperFileDirectly: true
                    ])
                }
            }
        }

    }
