pipeline {
    agent any

    tools{
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment{
        SCANNER_HOME=tool 'sonar-scanner'
    }
    
    stages {
        stage('Git-Checkout') {
            steps {
               git branch: 'main', changelog: false, credentialsId: 'a8f8d951-3c19-4107-9c87-e78391c69d96', poll: false, url: 'https://github.com/ASHUBHAIYA/Ekart.git'
            }
        }
         stage('Compile') {
            steps {
               sh "mvn compile"
            }
        }
        stage('OWASP-Check') {
            steps {
              dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'owasp'
              dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Sonar-Analysis') {
            steps{
             withSonarQubeEnv('Sonar-server') {
                sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Ekart \
                -Dsonar.java.binaries=. \
                -Dsonar.projectKey=Ekart  '''
                }
            }
        }
        stage('Build') {
            steps {
               sh "mvn clean package -DskipTests=true"
            }
        }
         
        stage('Docker-Image') {
            steps {
                withDockerRegistry(credentialsId: '1dc12a5a-be8b-49f1-9c4a-d443b73b59c3', url: '' ) {
                    sh 'docker build -t ekart -f docker/Dockerfile .'
                    sh 'docker tag ekart abhishek791996/ekart:latest'
                    sh 'docker push abhishek791996/ekart:latest'
                
                }
            }
        }
        stage('Docker-Deploy') {
            steps {
              withDockerRegistry(credentialsId: '1dc12a5a-be8b-49f1-9c4a-d443b73b59c3', url: '' ) {
                sh 'docker run -d --name shop-shop -p 8070:8070  abhishek791996/ekart:latest'
                }
            }
        }
    }
}
