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
        stage(test){
            sh '''
                test -f build/index.html
                npm test
            '''
        }
    }
}