pipeline {
    agent any
        environment{
            AWS_DEFAULT_REGION = 'eu-north-1'
        }
   
    stages {
        // stage('Docker'){
        //     steps{
        //         sh 'docker build -t my-playwright .'
        //     }
        // }

 stage ('Deploy to AWS'){
            agent {
                docker{
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "-u root --entrypoint=''"
                }
            }
       
            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
            sh '''
                aws --version
                yum install jq -y
                LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefinition.revision')
                echo $LATEST_TD_REVISION
               aws ecs update-service --cluster LearnJenkinsApp-Cluster-Prod_1 --service LeaernJenkinsApp-Service-Prod --task-definition LearnJenkinsApp-TaskDefinition-Prod:$LATEST_TD_REVISION
               aws ecs wait services-stable --cluster LearnJenkinsApp-Cluster-Prod_1 --services LeaernJenkinsApp-Service-Prod
             '''
}       
               
            }
        }

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
       
        // stage('Tests') {
        //     parallel {
        //         stage('Unit Test') {
        //             agent {
        //                 docker {
        //                     image 'node:18-alpine'
        //                     reuseNode true
        //                 }
        //             }
        //             steps {
        //                 sh ''' 
        //                 test -f build/index.html
        //                 npm test
        //                 '''
        //             }
        //             post {
        //                 always {
        //                     junit 'jest-results/junit.xml'
        //                 }
        //             }
        //         }
                
        //         stage('e2e test') {
        //             agent {
        //                 docker {
        //                     image 'my-playwright'
        //                     reuseNode true
        //                 }
        //             }
        //             steps {
        //                 sh ''' 
                        
        //                 serve -s build &
        //                 sleep 10
        //                 npx playwright test --reporter=html
        //                 '''
        //             }
        //             post {
        //                 always {
        //                     publishHTML([
        //                         allowMissing: false,
        //                         alwaysLinkToLastBuild: false,
        //                         keepAll: false,
        //                         reportDir: 'playwright-report',
        //                         reportFiles: 'index.html',
        //                         reportName: 'Playwright HTML Report',
        //                         reportTitles: '',
        //                         useWrapperFileDirectly: true
        //                     ])
        //                 }
        //             }
        //         }
        //     }
        // }
        
        // stage('Deploy Stage') {
        //     agent {
        //         docker {
        //             image 'my-playwright'
        //             reuseNode true
        //         }
        //     }

       
        //     steps {
        //         sh ''' 
               
        //         netlify --version
        //         echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
        //         netlify status 
        //         netlify deploy --dir=build --json > deploy-output.json
        //         CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' deploy-output.json)
        //         npx playwright test --reporter=html
        //         '''
        //     }
        //     post {
        //         always {
        //             publishHTML([
        //                 allowMissing: false,
        //                 alwaysLinkToLastBuild: false,
        //                 keepAll: false,
        //                 reportDir: 'playwright-report',
        //                 reportFiles: 'index.html',
        //                 reportName: 'Staging E2E',
        //                 reportTitles: '',
        //                 useWrapperFileDirectly: true
        //             ])
        //         }
        //     }
        // }

        // stage('Deploy prod') {
        //     agent {
        //         docker {
        //             image 'my-playwright'
        //             reuseNode true
        //         }
        //     }

        //     environment {
        //         CI_ENVIRONMENT_URL = 'https://66d46f75fa719039cf628266--storied-blancmange-03b508.netlify.app'
        //     }

        //     steps {
        //         sh ''' 
        //         node --version
                
        //         netlify --version
        //         echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
        //         netlify status
        //         netlify deploy --dir=build --prod
        //         npx playwright test --reporter=html
        //         '''
        //     }
        //     post {
        //         always {
        //             publishHTML([
        //                 allowMissing: false,
        //                 alwaysLinkToLastBuild: false,
        //                 keepAll: false,
        //                 reportDir: 'playwright-report',
        //                 reportFiles: 'index.html',
        //                 reportName: 'Production E2E',
        //                 reportTitles: '',
        //                 useWrapperFileDirectly: true
        //             ])
        //         }
        //     }
        // }

    }
}