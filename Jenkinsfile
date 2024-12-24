pipeline{
    agent any
    tools {
        jdk 'jdk17'
        maven 'Maven3'
    }
    environment {
        SCANNER_HOME=tool 'Sonar-Scanner'
    }
    stages{
        stage ('clean Workspace'){
            steps{
                cleanWs()
            }
        }
        stage ('checkout scm') {
            steps {
              git branch: 'master', url: 'https://github.com/Harshit-cyber-bit/jpetstore-6.git'
            }
        }
        stage ('maven compile') {
            steps {
                sh 'mvn clean compile'
            }
        }
        stage ('maven Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('Sonar-Server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Petshop \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=Petshop '''
                }
            }
        }
        stage("quality gate"){
            steps {
                script {
                  waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-Token'
                }
           }
        }
        stage ('Build war file'){
            steps{
                sh 'mvn clean install -DskipTests=true'
            }
        }
        stage("OWASP Dependency Check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./ --format XML ', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage ("Build") { 
          steps
              { 
                 sh 'docker build -t jpetstore .'
 
              }
           }
       stage ("Push") {
          steps
              { 
                 withCredentials([usernamePassword(credentialsId: 'DockerCred', passwordVariable: 'dockerpasswd', usernameVariable: 'dockeruser')])  {
    
                 sh 'docker login -u ${dockeruser} -p ${dockerpasswd}'
                 sh 'docker tag jpetstore:latest venky3690/jpetstore:latest'
                 sh 'docker push venky3690/jpetstore:latest'
                }
                 
              }
           }
       stage('Run Docker container on Jenkins Agent') {
             
            steps 
             {
                 sh "docker run -dt -p 8085:8080 docker.io/venky3690/jpetstore:latest"
             }
        }
        stage("Deploy to Kubernetes Cluster"){
            steps
         {
            sh "kubectl apply -f /home/ubuntu/Deployment.yml"
         }
       } 
   }
}
