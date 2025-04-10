pipeline {
    agent any
    tools {
        maven 'maven3'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/aclockworkobi9/bank-app.git'
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
                    sh "${SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectName=Multitier -Dsonar.projectKey=Multitier -Dsonar.java.binaries=target"
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
                withMaven(globalMavenSettingsConfig: 'mysetting', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy -DskipTests=true"
                }
            }
        }
        stage('Docker Build Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t aclockworkobi9/bank:latest ."
                    }
                }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o image-report.html alockworkobi9/bank:latest"
            }
        }
        stage('Docker Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push aclockworkobi9/bank:latest"
                    }
                }
            }
        }
        // Add the Kubernetes interaction here
        stage('Kubernetes Interaction') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'project-cluster', contextName: '', credentialsId: 'k8s-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://D3B23B652BDBB67E1559EB9E0DB9EC62.gr7.us-east-1.eks.amazonaws.com') {
                    // some block
                    sh "kubectl apply -f ds.yaml -n webapps --validate=false"
                    sleep 30
                }
            }
        }
        stage('Verify deployment') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'project-cluster', contextName: '', credentialsId: 'k8s-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://D3B23B652BDBB67E1559EB9E0DB9EC62.gr7.us-east-1.eks.amazonaws.com') {
                    // some block
                    sh "kubectl get pods -n webapps"
                    sh "kubectl get svc -n webapps"
                }
            }
        }
    }
}
