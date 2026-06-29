pipeline{
    agent none

    environment{
        NETLIFY_SITE_ID = '4a55c6ac-41cb-4cdc-b5c4-3d54ae962a1b'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages{

        stage ('Build'){

            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            steps{
                sh '''
                    echo "트리거 테스트 " 
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }

        }

        stage ('AWS'){
            agent {
                docker {
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "--entrypoint=''"
                }
            }
            environment{
                AWS_S3_BUCKET = 'learn-jenkins-20260629'
            }

            steps {
                withCredentials([usernamePassword(credentialsId: 'my_aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        aws s3 sync build s3://$AWS_S3_BUCKET
                    '''
                }
                
            }

        }

        stage ('Test'){
            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                }
            }
            steps{
                echo 'Test stage'
                sh '''
                    test -f build/index.html
                    npm test
                '''
                
            }
        }
        stage('E2E'){
            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                }
            }

            steps{
                sh'''
                    npm install serve
                    node_modules/.bin/serve -s build & sleep 10
                    npx playwright test --reporter=html
                '''
            }

        }
        stage('Deploy staging'){
            agent{
                docker{
                    image 'node:18-bullseye'
                }
            }

            steps{
                sh '''
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify deploy --dir=build
                '''
            }
        }

        stage('Approval'){
            agent none 

            steps{
                timeout(1) {
                    input message: '운영환경에 배포할까요? ', ok: '네, 배포합니다.'
                }
            }
            
            
        }

        stage('Deploy prod'){
            agent{
                docker{
                    image 'node:18-bullseye'
                }
            }
            steps{
                sh '''
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version
                    echo "배포중 사이트 아이디 : $4a55c6ac-41cb-4cdc-b5c4-3d54ae962a1b"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }

        stage ('Prod E2E'){
            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                }
            }

            environment{
                CI_ENVIRONMENT_URL = 'https://benevolent-cascaron-4fc37a.netlify.app'
            }

            steps{
                sh '''
                    npx playwright test --reporter=html
                '''
            }
        }
    }

    post {
        always {
            junit 'jest-results/junit.xml'
        }
    }
}