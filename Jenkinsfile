pipeline {
    agent any 
    environment {
        registryCredential = 'docker'
        imageName = 'mannyaboah/nodefrontend'
        dockerImage = ''
        }
    stages {
        stage('Setting Up') {
             agent {
                docker { 
                    image 'node:14-alpine'
                    args '-e HOME=/tmp -e NPM_CONFIG_PREFIX=/tmp/.npm'
                    reuseNode true
                }
            }
            steps {
                echo 'Retrieve source from github. run npm install and npm test'
                git branch: 'manny-jenkins',
                    url: 'https://github.com/isMunim/AdvDevOpsLearning-Frontend.git'
                echo 'checking if that worked'
                sh 'ls -a'
                sh 'npm install'
                sh 'npm test'
            }
        }
        stage('Building image') {
            steps{
                script {
                    echo 'build the image' 
                    dockerImage = docker.build imageName
                }
            }
            }
        stage('Push Image') {
            steps{
                script {
                    echo 'push the image to docker hub' 
                    docker.withRegistry('', registryCredential) {
                        dockerImage.push("$BUILD_NUMBER")
                    }
                    
                }
            }
        }     
         stage('deploy to k8s') {
             agent {
                docker { 
                    image 'google/cloud-sdk:latest'
                    args '-e HOME=/tmp'
                    reuseNode true
                        }
                    }
            steps {
                echo 'connecting to the GKE cluster'
                sh 'gcloud container clusters get-credentials manny-cluster --zone us-central1-a --project dtc-102021-u106'
                
                echo 'set image to update the container'
                sh 'kubectl set image deployment/nodefrontend nodefrontend=$imageName:$BUILD_NUMBER'
            }
        }     
        stage('Remove local docker image') {
            steps{
                sh "docker rmi $imageName:latest"
                sh "docker rmi $imageName:$BUILD_NUMBER"
            }
        }
    }
}