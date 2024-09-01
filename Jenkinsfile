pipeline {
    agent any

    environment{
        NETLIFY_SITE_ID= 'f2be4442-6706-4c54-9266-de41ce6d6112'
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
                stage ('Deploy'){
                    agent{
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                    }
                }
                    steps{
                        sh '''
                        npm install netlify-cli
                        node_modules/.bin/netlify --version
                        echo "Deployng to production. Site ID: $NETLIFY_SITE_ID"
                        '''
                    }
                }
    }
}