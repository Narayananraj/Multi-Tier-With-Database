pipeline {
    agent any
    
    tools {
        jdk 'jdk17'          // Ensure JDK 17 is installed and configured in Jenkins
        maven 'maven3'        // Ensure Maven 3 is installed and configured in Jenkins
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Narayananraj/Multi-Tier-With-Database.git'
            }
        }
        stage('Compile') {
            steps {
               sh "mvn compile"
            }
        }
         stage('Test') {
            steps {
               sh "mvn test -DskipTests=true"
            }
        }
         stage('Trivy FS Scan') {
            steps {
                sh "trivy fs --format table -o fs-report.html ."
            }
        }
         stage('SonarQube Analysis') {
            steps {
               withSonarQubeEnv('sonar') {
                    sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Multitier -Dsonar.projectKey=Multitier -Dsonar.java.binaries=target/classes"
                   
            }
        
        }
      }
       stage('Build') {
            steps {
               sh "mvn package -DskipTests=true"
            }
        }
         stage('Publish to Nexus') {
            steps {
               withMaven(globalMavenSettingsConfig: 'settings-maven', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {

                   sh "mvn deploy -DskipTests=true"
               }
            }
        }
         stage('Docker Build') {
            steps {
                script {
                    // Build Docker image
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker build -t narayananraj/bankapp:latest ."
                    }
                }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o fs-report.html narayananraj/bankapp:latest"
            }
        }
         stage('Docker Push Image') {
            steps {
                script {
                    // Build Docker image
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker push  narayananraj/bankapp:latest"
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
               withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://144131E476369B4F222BD73912C92B88.gr7.eu-west-2.eks.amazonaws.com') {
                    sh "kubectl apply -f ds.yml --dry-run=client"
                    sh "kubectl apply -f ds.yml -n webapps"
                    sleep 30
                   
               }
            }
        }
         stage('Verify Deployment') {
            steps {
               withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://144131E476369B4F222BD73912C92B88.gr7.eu-west-2.eks.amazonaws.com') {
                    sh "kubectl get pods -n webapps"
                    sh "kubectl get svc -n webapps"
                   
               }
            }
        }
}
}
