pipeline{
     agent any
     
     tools{
         jdk 'jdk17'
         nodejs 'node16'
     }
     environment {
         SCANNER_HOME=tool 'sonarqube-scanner'
     }
     
     stages {
         stage('Clean Workspace'){
             steps{
                 cleanWs()
             }
         }
         stage('Checkout from Git'){
             steps{
                 git branch: 'main', url: 'https://github.com/Ashfaque-9x/a-swiggy-clone.git'
             }
         }
         stage("Sonarqube Analysis "){
             steps{
                 withSonarQubeEnv('SonarQube-Server') {
                     sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Swiggy-CI \
                     -Dsonar.projectKey=Swiggy-CI '''
                 }
             }
         }
         stage("Quality Gate"){
            steps {
                 script {
                     waitForQualityGate abortPipeline: false, credentialsId: 'SonarQube-Token' 
                 }
             } 
         }
         stage('Install Dependencies') {
             steps {
                 sh "npm install"
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
                    withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker'){   
                        sh "docker build -t swiggy-clone ."
                        sh "docker tag swiggy-clone anildevops23/swiggy-clone:latest "
                        sh "docker push anildevops23/swiggy-clone:latest "
                     }
                 }
             }
         }
         stage("TRIVY"){
             steps{
                 sh "trivy image ashfaque9x/swiggy-clone:latest > trivyimage.txt" 
             }
         }
          stage('Deploy to Kubernets'){
             steps{
                 script{
                     dir('Kubernetes') {
                         withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: '', contextName: '', credentialsId: 'kubernetes', namespace: '', serverUrl: '']]) {
                         sh 'kubectl delete --all pods'
                         sh 'kubectl apply -f deployment.yml'
                         sh 'kubectl apply -f service.yml'
                         }   
                     }
                 }
            }
        }
     }
}
