pipeline {
    agent any
    
    tools {
        maven 'maven3'
        jdk 'jdk17'
    }
    
    environment {
        SCANNER_HOME= tool 'sonar-scanner' 
    }

    stages {
        stage('Git Checkout') {
            steps {
                git 'https://github.com/arifsadiq/DevOps-EKART.git'
            }
        }
        stage('Maven Compile') {
            steps {
                sh 'mvn clean compile'
            }
        }
        stage('Maven Test') {
            steps {
                sh 'mvn test -DskipTests=true'
            }
        }
        stage('SonarQube Scanner') {
            steps {
                withSonarQubeEnv('sonar') {
                  sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Ekart-proj \
                    -Dsonar.projectKey=Ekart-proj -Dsonar.java.binaries=. '''
                }
            }
        }
        stage('OWASP Dependency-check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DC'
                  dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests=true'
            }
        }
        stage('Deploy Artifact to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                  sh 'mvn deploy -DskipTests=true'
                }
            }
        }
        stage('Docker Build & Tag') {
            steps {
                script{
                  withDockerRegistry(credentialsId: 'docker-cred1', toolName: 'docker') {
                    sh 'docker build -t ari786/ekart:latest -f docker/Dockerfile .'
                  }
                }
            }
        }
        stage('Trivy Scan') {
            steps {
                sh 'trivy image ari786/ekart:latest > trivy-scanreport.txt'
            }
        }
        stage('Docker Push') {
            steps {
                script{
                  withDockerRegistry(credentialsId: 'docker-cred1', toolName: 'docker') {
                    sh 'docker push ari786/ekart:latest'
                  }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script{
                  withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://192.168.1.200:6443') {  
                    sh 'kubectl apply -f deploymentservice.yml -n webapps'
                    sh 'kubectl get pods -o wide -n webapps'
                    sh 'kubectl get svc -n webapps'
                      
                  }
                
                }
            }
        }
         
    }
}
