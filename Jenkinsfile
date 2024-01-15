pipeline {
    agent any

    tools {
        maven 'maven3'
        jdk 'jdk-17'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/imHarry404/Ekart.git'
            }
        }

        stage('Compile the code') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'mvn test -DskipTests=true'
                // If the test fails, it will skip
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=EKART -DsonarprojectName=EKART\
                    -Dsonar.java.binaries=.'''
                }
            }
        }

        stage('OWASP Dependancy Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                
                
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn package -DskipTests=true'
            }
        }
        
        stage('Deploy to Nexus') {
            steps {
                // deploying the artificats to Nexus
                withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk-17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                  sh 'mvn deploy -DskipTests=true'
                }
            }
        }
        
        stage('Build & Tag Image') {
            steps {
                script {
                    // This step should not normally be used in your script. Consult the inline help for details.
                    withDockerRegistry(credentialsId: 'docker-credentials', toolName: 'docker') {
                            sh "docker build -t imharry404/ekart:latest -f docker/Dockerfile ."
                    }
                    
                }
            }
        }
        
        stage('Trivy Scan') {
            steps {
                sh "trivy image imharry404/ekart:latest > trivy-report.txt"
            }
        }
        
         stage('Push Image') {
            steps {
                script {
                    
                    withDockerRegistry(credentialsId: 'docker-credentials', toolName: 'docker') {
                            sh "docker push imharry404/ekart:latest"
                    }
                }
            }
        }
        
        stage('Deploy to k8s') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.39.201:6443') {
                sh "kubectl apply -f deploymentservice.yml -n webapps"
                sh "kubectl get svc -n webapps"
                }
            }
        }
        
        stage('Last') {
            steps {
                echo "ho gya bhai....bye ab"
            }
        }
        
    }
}
