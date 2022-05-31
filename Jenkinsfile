pipeline {
    agent any 
   environment {
        registryCredential = 'docker'
        imageName = 'gkoshiro/internal'
        dockerImage = ''
        }
    stages {
        stage('Run the tests') {
             agent {
                docker { 
                    image 'node:14-alpine'
                    args '-e HOME=/tmp -e NPM_CONFIG_PREFIX=/tmp/.npm'
                    reuseNode true
                }
            }
            steps {
                echo 'Retrieving source from github' 
                git branch: 'main',
                    url: 'https://github.com/kazzuyak/events-app-internal'
                echo 'Did we get the source?' 
                sh 'ls -a'
                echo 'install dependencies' 
                sh 'npm install'
                echo 'Run tests'
                sh 'npm test'
                echo 'Tests passed on to build and deploy Docker container'
            }
        }
        stage('Building image') {
            steps{
                script {
                    dockerImage = docker.build imageName
                }
            }
            }
            stage('Deploy Image') {
            steps{
                script {
                docker.withRegistry( '', registryCredential ) {
                    dockerImage.push("$BUILD_NUMBER")
                }
                }
            }
        }     
         stage('get k8s credentials') {
             agent {
                docker { 
                    image 'google/cloud-sdk:latest'
                    args "-e HOME=${WORKSPACE}/kube/"
                    reuseNode true
                    }
                    }
            steps {
                sh "rm -rf ${WORKSPACE}/kube/"
                echo 'Get cluster credentials'
                sh 'gcloud container clusters get-credentials cluster-1 --zone us-central1-c --project roi-qt-may22-u118'
            }
        }     
         stage('update k8s') {
             agent {
                docker { 
                    image 'google/cloud-sdk:latest'
                    args '-e HOME=/tmp'
                    reuseNode true
                        }
                    }
            steps {
                echo 'Set the image'
                     sh "kubectl --kubeconfig=${WORKSPACE}/kube/.kube/config set image deployment/demo-api demo-api=${env.imageName}:${env.BUILD_NUMBER}"
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
