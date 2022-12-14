pipeline {
    agent any

    environment {
        registry = "267753450809.dkr.ecr.ap-south-1.amazonaws.com/ltirepo"
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/shwetak1319/CICDusingKubernetes.git']]])
            }
        }
        
        stage('Build Jar') {
            steps {
                sh "mvn clean install"
            }
        }
    
        stage ("Build image") {
            steps {
                script {
                    sh "docker build -t ltirepo ."
                    sh "docker tag ltirepo:latest ${env.registry}:0.0.${env.BUILD_NUMBER}"
                }
            }
        }
        
        stage ("docker push") {
            steps {
                script {
                    sh "aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 267753450809.dkr.ecr.ap-south-1.amazonaws.com"
                    sh "docker push ${env.registry}:0.0.${env.BUILD_NUMBER}" 
                }
            }   
        }
        stage ("Kube Deploy") {
            steps {
                withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'eks', namespace: '', serverUrl: '') {
                        sh "sed -i s/_BUILD_NUMBER_/${env.BUILD_NUMBER}/g eks-deploy-from-ecr.yaml"
                        sh "kubectl apply -f eks-deploy-from-ecr.yaml"
                    }
                }
            }
        }
    }
