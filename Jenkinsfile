pipeline {
    agent any

    environment{
        NETLIFY_SITE_ID= 'f2be4442-6706-4c54-9266-de41ce6d6112'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        CI_ENVIRONMENT_URL= 'https://66d46f75fa719039cf628266--storied-blancmange-03b508.netlify.app/'

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
                stage ('Deploy Stage'){
                    agent{
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                    }
                }
                    steps{
                        sh '''
                        echo 'Small change'
                        npm install netlify-cli
                        node_modules/.bin/netlify --version
                        echo "Deployng to staging. Site ID: $NETLIFY_SITE_ID"
                        node_modules/.bin/netlify status
                        node_modules/.bin/netlify deploy --dir=build
                        '''
                    }
                }

                stage('Approval'){
                    steps{
                        timeout(time: 15, unit: 'MINUTES'){
                            input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure' 
                        }
                        
                    }
                }
                stage ('Deploy Prod'){
                    agent{
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                    }
                }
                    steps{
                        sh '''
                        echo 'Small change'
                        npm install netlify-cli
                        node_modules/.bin/netlify --version
                        echo "Deployng to production. Site ID: $NETLIFY_SITE_ID"
                        node_modules/.bin/netlify status
                        node_modules/.bin/netlify deploy --dir=build --prod
                        '''
                    }
                }
        stage('PROD e2e') {
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
                        reportName: 'Playwright E2E',
                        reportTitles: '',
                        useWrapperFileDirectly: true
                    ])
                }
            }
        }

    }
}