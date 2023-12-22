pipeline{
    agent any

    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }

    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages{
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }

        stage('Checkout from Git'){
            steps{
                git branch: 'main', credentialsId: 'git-Pass', url: 'https://github.com/AASAITHAMBI57/Netflix-clone-aas.git'
            }
        }

        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix '''
                }
            }
        }

        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
                }
            } 
        }

        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }

        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }

        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
                       sh "docker build --build-arg TMDB_V3_API_KEY=96b8030c856125d846b5006527898552 -t netflix ."
                       sh "docker tag netflix aasaithambi5/netflix:latest "
                       sh "docker push aasaithambi5/netflix:latest "
                    }
                }
            }
        }

        stage("TRIVY"){
            steps{
                sh "trivy image aasaithambi5/netflix:latest > trivyimage.txt" 
            }
        }

        stage('Remove Previous Container'){
            steps{
                sh 'docker rm -f netflix'
            }
        }

        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name netflix -p 8081:80 aasaithambi5/netflix:latest'
            }
        }

        stage('Deploy to kubernets'){
            steps{
                script{
                    dir('Kubernetes') {
                        withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'Kube-Pass', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                            sh 'kubectl apply -f deployment.yml'
                            sh 'kubectl apply service.yml'
                        }
                    }
                }
            }
        }
    }
}  
