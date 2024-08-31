pipeline{
    agent any
    stages{
        stage('Build'){
            agent{
                docker{
                    image "node:18-Alpine"
                    reuseNode true
                }
            }
            steps{
                sh '''
                    npm --version
                '''
            }
        }
    }
}