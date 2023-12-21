pipeline{
    agent any


    stages{
        stage('Checkout from Git'){
            steps{
                git branch: 'main', credentialsId: 'git-Pass', url: 'https://github.com/AASAITHAMBI57/Netflix-clone-aas.git'
            }
        }
    }
}
