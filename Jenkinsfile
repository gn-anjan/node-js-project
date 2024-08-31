pipeline{
    agent any
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
        stage('Test'){
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
                    junit "test-results/junit.xml"
                }
            }
        }
        stage('E2E Testing'){
            agent{
                docker{
                    image "mcr.microsoft.com/playwright:v1.46.1-jammy"
                    reuseNode true
                }
            }
            steps{
                sh '''
                    npm install serve
                    node_modules/.bin/serve -s build
                    npx playwrigth test
                '''
            }
            post{
                always{
                    junit "test-results/junit.xml"
                }
            }
        }
    }
}