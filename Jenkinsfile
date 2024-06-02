pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        stage('git checkout') {
            steps {
                echo 'getting code from git...'
                git branch: 'main', changelog: false, credentialsId: 'git-cred', poll: false, url: 'https://github.com/mallikharjun999/CI-CD-APP-.git'
            }
        }
        stage('compile the code') {
            steps {
                echo 'compiling the code...'
                sh "mvn compile"
            }
        }
        stage('Test') {
            steps {
                echo 'testing code...'
                sh "mvn test -DskipTests=true"
            }
        }
        stage('trivy scan file system') {
            steps {
                echo 'scanning with trivy...'
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        stage('sonarqube analysis') {
            steps {
                echo 'sourcecode analysis with sonar...'
                withSonarQubeEnv(installationName: 'sonar', credentialsId: 'sonar-token') {
                sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=cicd-app -Dsonar.projectName=cicd-app \
                    -Dsonar.java.binaries=.'''
                }
            }     
        }
        stage('Build') {
            steps {
                echo 'Building the package'
                sh "mvn package -DskipTests=true"
            }
        }
        stage('deploy artifacts to nexus') {
            steps {
                echo 'deploying artifacts to nexus'
                withMaven(globalMavenSettingsConfig: 'maven-setting', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy -DskipTests=true"
                }
            }
        }
        stage('Build & tag docker image') {
            steps {
                echo 'Building the docker image'
                script {
                    withDockerRegistry(credentialsId: 'dockerreg-cred', toolName: 'docker') {
                        sh "docker build -t mallikharjun999/cicd-app:latest ."
                    }       
                }
            }
        }
        stage('trivy scan docker image') {
            steps {
                echo 'scanning docker image with trivy...'
                sh "trivy image --format table -o trivy-image-report.html mallikharjun999/cicd-app:latest"
            }
        }
        stage('publish docker image to docker hub') {
            steps {
                echo 'pushing the docker image'
                script {
                    withDockerRegistry(credentialsId: 'dockerreg-cred', toolName: 'docker') {
                        sh "docker push  mallikharjun999/cicd-app:latest"
                    }       
                }
            }
        }
        stage ('deploy the app to kubernetes'){
            steps {
                echo 'deploying the app to production k8s'
                withKubeConfig(caCertificate: '', clusterName: 'AM-EKS', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://3C1AD7E1D7D693E59350400380D8B746.sk1.us-east-2.eks.amazonaws.com') {
                    sh "kubectl apply -f K8Deployment.yml -n webapps"
                    sleep 60
                }
            }
        }
        stage ('verifying the deployment'){
            steps {
                echo 'verifying the app on k8s'
                withKubeConfig(caCertificate: '', clusterName: 'AM-EKS', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://3C1AD7E1D7D693E59350400380D8B746.sk1.us-east-2.eks.amazonaws.com') {
                    sh "kubectl get pods -n webapps"
                    sh "kubectl get svc -n webapps"
                }
            }
        }
        
        
    }
    
}
