pipeline {
    agent any
    environment {
        registryCredential = 'dockerhub'
        imageName = 'webashu/internal'
        dockerImage = ''
    }   
    stages {
        stage("Run the test") {
            agent {
                docker { 
                    image 'node:14-alpine'
                    args '-e HOME=/tmp -e NPM_CONFIG_PREFIX=/tmp/.npm'
                    reuseNode true
                }
            }
            steps {
                echo "checkout source code"
                git branch: 'master', url: 'https://github.com/TheAshishRepo/devops_internal.git'
                echo "building ..."
                sh "npm install"
                echo "testing...."
                sh "npm test"
            }

		}
		stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    // sh 'mvn clean package sonar:sonar'
                    sh '''mvn sonar:sonar -Dsonar.host.url=http://34.139.94.60:9000 \
                          -Dsonar.login=b1d1bfa64b8d3b90790432a0008ca203b6c6cc11'''
                }
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
