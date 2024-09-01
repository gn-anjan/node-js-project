pipeline{
    agent any
    environment{
        netlify_site_id = '4cb1313b-ad9b-4f82-8e19-87d2131cd59b'
        netlify_auth_token = credentials('netlify_login')
    }
    stages{
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
                            npx playwright test
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
        stage("Deploy"){
            agent{
                docker{
                    image "node:18-alpine"
                }
            }
            steps{
                // install netlify
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo $netlify_site_id
                    nelify login
                    // typo fixed
                    node_modules/.bin/netlify status
                '''
            }
        }
        
    }
}