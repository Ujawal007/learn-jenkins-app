pipeline {
    agent any

    environment{
        // NETLIFY_SITE_ID = '59e42f00-508f-4bd3-b450-e9e75acd2c5c'
        // NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        APP_NAME = 'learnjenkinsapp'
        AWS_DEFAULT_REGION = 'us-east-1'
        AWS_DOCKER_REGISTRY = '903814928721.dkr.ecr.us-east-1.amazonaws.com'
        AWS_ECS_CLUSTER = 'LearnJenkinsApp-Cluster-prod'
        AWS_ECS_SERVICE_PROD = 'LearnJenkinsApp-Service-Prod'
        AWS_ECS_TD_PROD = 'LearnJenkinsApp-TaskDefinition-Prod'
    }

    stages {

        stage('Build') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh'''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Build Docker image') {
            agent{
                docker{
                    image 'my-aws-cli'
                    reuseNode true
                    args "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
                }
            }
            steps{
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh'''
                        docker build -t $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION .
                        aws ecr get-login-password | docker login --username AWS --password-stdin $AWS_DOCKER_REGISTRY
                        docker push $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION
                    '''
                }
            }
        }

        stage('Deploy to AWS'){
            agent{
                docker{
                    image 'my-aws-cli'
                    reuseNode true
                    args "--entrypoint=''"
                }
            }
            // environment{
            //     AWS_S3_BUCKET = 'learn-jenkins-20241029'
            // }
            steps{
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh'''
                        aws --version
                        sed -i "s/#APP_VERSION#/$REACT_APP_VERSION/g" aws/task-definition-prod.json
                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefinition.revision')
                        aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE_PROD --task-definition $AWS_ECS_TD_PROD:$LATEST_TD_REVISION
                        aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --services $AWS_ECS_SERVICE_PROD
                    '''
                }
            }
        }

        // stage('Tests') {
        //     parallel{
        //         stage('Unit Tests'){
        //             agent{
        //                 docker{
        //                     image 'node:18-alpine'
        //                     reuseNode true
        //                 }
        //             }
        //             steps{
        //                 sh'''
        //                     #test -f build/index.html
        //                     npm test
        //                 '''
        //             }
        //             post{
        //                 always{
        //                     junit 'jest-results/junit.xml'
        //                 }
        //             }
        //         }

        //         stage('E2E'){
        //             agent{
        //                 docker{
        //                     image 'my-playwright'
        //                     reuseNode true
        //                 }
        //             }
        //             steps{
        //                 sh'''
        //                     serve -s build &
        //                     sleep 10
        //                     npx playwright test  --reporter=html
        //                 '''
        //             }
        //             post{
        //                 always{
        //                     publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Local E2E', reportTitles: '', useWrapperFileDirectly: true])
        //                 }
        //             }
        //         }
        //     }
        // }

        // stage('Deploy staging') {
        //     agent{
        //         docker{
        //             image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
        //             reuseNode true
        //         }
        //     }

        //     environment{
        //         CI_ENVIRONMENT_URL = 'STAGING_URL_TO_BE_SET'
        //     }
            
        //     steps {
        //         sh'''
        //             npm install netlify-cli node-jq
        //             node_modules/.bin/netlify --version
        //             echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
        //             node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
        //             CI_ENVIRONMENT_URL=$(node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json)
        //             npx playwright test  --reporter=html
        //         '''
        //         post{
        //             always{
        //                 publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
        //             }
        //         }
        //     }
            
        // }

        // stage('Deploy staging') {
        //     agent{
        //         docker{
        //             image 'my-playwright'
        //             reuseNode true
        //         }
        //     }
        //     steps {
        //         sh'''
        //             netlify --version
        //             echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
        //             netlify deploy --dir=build --json > deploy-output.json
        //         '''
        //         script{
        //             env.STAGING_URL = sh(script: "jq -r '.deploy_url' deploy-output.json", returnStdout: true)
        //         }
        //     }
            
        // }

        // stage('Staging E2E'){
        //     agent{
        //         docker{
        //             image 'my-playwright'
        //             reuseNode true
        //         }
        //     }
        //     environment{
        //             CI_ENVIRONMENT_URL = "${env.STAGING_URL}"
        //     }
        //     steps{
        //         sh'''
        //             npx playwright test --reporter=html
        //         '''
        //     }
        //     post{
        //         always{
        //             publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
        //         }
        //     }
        // }

        // stage('Deploy prod') {
        //     agent{
        //         docker{
        //             image 'my-playwright'
        //             reuseNode true
        //         }
        //     }
        //     environment{
        //         CI_ENVIRONMENT_URL = 'https://cerulean-babka-e8a77b.netlify.app/'
        //     }
        //     steps {
        //         sh'''
        //             node --version
        //             netlify --version
        //             echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
        //             netlify status
        //             netlify deploy --dir=build --prod
        //             npx playwright test --reporter=html
        //         '''
        //     }
        //     post{
        //         always{
        //             publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E', reportTitles: '', useWrapperFileDirectly: true])
        //         }
        //     }
        // }

    }

}
