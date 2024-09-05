pipeline{
    agent any
    environment{
        NETLIFY_SITE_ID = '5855610f-46ba-4a01-a5d1-b04ec5365fc1'
        NETLIFY_AUTH_TOKEN = credentials('netlify_token')
    }
    stages{
        // test
        stage('Build'){
            agent{
                docker{
                    image "node:18-alpine"
                    reuseNode true
                }
            }
            steps{
                sh '''
                    npm install
                    npm ci
                    npm run build
                '''
            }
        }
        stage("Tests"){
            parallel{
                stage('Unit Test'){
                    agent{
                        docker{
                            image "node:18-alpine"
                            reuseNode true
                        }
                    }
                    steps{
                        sh '''
                            test -f build/index.html
                            npm test
                        '''
                    }
                    post{
                        always{
                            junit "jest-results/junit.xml"
                        }
                    }
                }
                stage('E2E Testing'){
                    agent{
                        docker{
                            image "mcr.microsoft.com/playwright:v1.39.0-jammy"
                            reuseNode true
                        }
                    }
                    steps{
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build & 
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                }

            }
            post{
                always{
                    junit "jest-results/junit.xml"
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }                        
        }
        stage("Deploy to stage"){
            agent{
                docker{
                    image "node:18-alpine"
                    reuseNode true
                }
            }
            steps{
                // install netlify
                sh '''
                    npm install netlify-cli node-jq
                    node_modules/.bin/netlify --version
                    echo $NETLIFY_SITE_ID
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --json > stage_deploy.json
                    node_modules/.bin/node-jq -r '.deploy_url' stage_deploy.json
                '''
            }
            script {
                env.staging_url = sh(script: "node_modules/.bin/node-jq -r '.deploy_url' stage_deploy.json", returnStdout: true)
            }
        stage('Stage E2E'){
            agent{
                docker{
                    image "mcr.microsoft.com/playwright:v1.39.0-jammy"
                    reuseNode true
                }
            }
            environment{
                CI_ENVIRONMENT_URL = "${env.staging_url}"
            }
            steps{
                sh '''
                    npx playwright test --reporter=html
                '''
            }
            post{
                always{
                    junit "jest-results/junit.xml"
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
        }
        stage('Approval'){      
            steps{
                timeout(time: 20, unit: 'SECONDS')
                input 'Ready for production?'
            }
        }
        stage("Deploy to prod"){
            agent{
                docker{
                    image "node:18-alpine"
                    reuseNode true
                }
            }
            steps{
                // install netlify
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo $NETLIFY_SITE_ID
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }
        stage('Prod E2E'){
            agent{
                docker{
                    image "mcr.microsoft.com/playwright:v1.39.0-jammy"
                    reuseNode true
                }
            }
            environment{
                CI_ENVIRONMENT_URL = 'https://keen-melba-b1c4b0.netlify.app'
            }
            steps{
                sh '''
                    npx playwright test --reporter=html
                '''
            }
            post{
                always{
                    junit "jest-results/junit.xml"
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
        
    }
}