pipeline{
    agent any
    stages{
        stage('Build'){
            agent{
                docker{
                    image "Node:18-Alpine"
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