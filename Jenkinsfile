pipeline {
    agent any
    
    tools{
        maven 'maven3'
        jdk 'jdk17'
    }
    environment{
        SCANNER_HOME= tool 'Sonar-scanner'
    }

    stages {
        stage('git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/mintu1/Expenses-Tracker-WebApp.git'
            }
        }
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test -DskipTests=true'
            }
        }
        stage('Sonarqube Analysis') {
            steps {
                withSonarQubeEnv('Sonar') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=santa \
                    -Dsonar.projectKey=santa -Dsonar.java.binaries=. '''
                }
            }
        }
        stage('OWASP scan') {
            steps {
                dependencyCheck additionalArguments: '--scan . ', odcInstallation: 'OWASP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('build app') {
            steps {
                sh 'mvn package -DskipTests=true'
            }
        }
        stage('Docker login') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker build -t santa:latest .'
                        echo 'login successfull'
                    }
                }
            }
        }
        stage('tag and push Docker Image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker tag santa:latest amitk4452/santa:latest'
                        sh 'docker push amitk4452/santa:latest'
                    }
                }
            }
        }
        stage('Deploy on docker') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker run -d -p 8081:8080 --name santa-app amitk4452/santa:latest'
                    }
                }
            }
        }
        stage('Trivy Scan') {
            steps {
                sh 'Trivy image amitk4452/santa:latest'
            }
        }
    }
}
