pipeline {
    agent any
    environment {
        registryCredential = 'DockerHub'
        imageName = 'vkuppuswamy/internal'
        dockerImage = ''
    }   
    stages {
        stage("Run the test") {
            agent {
                docker { 
                    image '18-alpine3.15'
                    args '-e HOME=/tmp -e NPM_CONFIG_PREFIX=/tmp/.npm'
                    reuseNode true
                }
            }
            steps {
                echo "checkout source code"
                git branch: 'main', credentialsId: 'Git', url: 'https://github.com/TheAshishRepo/devops_internal.git'
                echo "building ..."
                sh "npm install"
                echo "testing...."
                sh "npm test"
            }

        }
        stage('Build image') {
            steps {
                script {
                    echo 'Starting to build docker image'
                    dockerImage = docker.build ("${env.imageName}:${env.BUILD_ID}")
                }
            }
        }
        stage('Push image') {
            steps {

               script {
                docker.withRegistry( '', registryCredential ) {
                dockerImage.push("${env.BUILD_ID}")
                }       
            }
         }
        }
        stage("deploy to k8s") {
            agent {
                docker {
                    image 'google/cloud-sdk:latest'
                    args '-e HOME=/tmp'
                    reuseNode true
                }
            }
            steps {
                echo "get cluster credentials"
                sh "gcloud container clusters get-credentials demo-cluster --zone us-east1-b --project theta-shuttle-357004"
                sh "kubectl set image deployment/internal-deployment internal=${env.imageName}:${env.BUILD_ID} --namespace=terraform"
            }
        }
        stage('Remove local docker image') {
            steps{
                sh "docker rmi ${env.imageName}:${env.BUILD_ID}"
            }
        }
    }
}
