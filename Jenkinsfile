pipeline{
    agent any
    stages{
        stage('Jenkis Discovery'){
            steps{
                echo 'Jankins discovery interface'
            }
        }


        stage('Build'){
            steps{
                sh 'mvn clean package'
            }
        }

        stage('Build Image'){

            steps{
                script{
                    discovery = docker.build("c0d8/discovery:${env.BUILD_ID}", "-f Dockerfile .")
                }
            }

        }

        stage('Push Image'){
            steps{
                script{
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credential'){
                        discovery.push("${env.BUILD_ID}")
                        discovery.push("latest")
                    }
                }
            }
        }
       
    }
}