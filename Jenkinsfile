pipeline {
    agent any
    environment {
        K8S_PORT = 53527
        TARGET = 'local'
    }
    stages {
        stage('Build Auth') {
            steps {
                build job: 'classroom.auth', wait: true
            }
        }
        stage('Build') { 
            steps {
                sh 'mvn clean package'
            }
        }      
        stage('Build Image') {
            steps {
                script {
                    image = docker.build("c0d8/gateway:${env.BUILD_ID}", "--platform linux/amd64 -f Dockerfile .")
                }
            }
        }
        stage('Push Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credential') {
                        image.push("${env.BUILD_ID}")
                        image.push("latest")
                    }
                }
            }
        }
        stage('Deploy on Local K8s') {
            when { 
                environment name: 'TARGET', value: 'local' 
            }
            steps {
                withCredentials([ string(credentialsId: 'minikube-credential', variable: 'api_token') ]) {
                    sh "kubectl --token $api_token --server https://host.docker.internal:${env.K8S_PORT}  --insecure-skip-tls-verify=true apply -f ./k8s/deployment.yaml"
                    sh "kubectl --token $api_token --server https://host.docker.internal:${env.K8S_PORT}  --insecure-skip-tls-verify=true apply -f ./k8s/service.yaml"
                }
            }
        }
        stage('Deploy on AWS K8s') {
            when { 
                environment name: 'TARGET', value: 'aws' 
            }
            steps {
                withCredentials([ string(credentialsId: 'minikube-credential', variable: 'api_token') ]) {
                    sh "kubectl apply -f ./k8s/deployment.yaml"
                    sh "kubectl apply -f ./k8s/service.yaml"
                }
            }
        }

    }
}