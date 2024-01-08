pipeline {
    environment {
        dockerImageName = "thetips4you/nodeapp"
        awsAccountId = "866762610186"
        awsRegion = "us-east-1"
        ecrRepository = "866762610186.dkr.ecr.us-east-1.amazonaws.com/kube-node"
    }

    agent any

    stages {
        stage('Checkout Source') {
            steps {
                sh 'echo passed'
                //git branch: 'main' url: 'https://github.com/deepalekhak/kube-node.git'
            }
        }

        stage('Build Image') {
            steps {
                script {
                    dockerImage = docker.build dockerImageName
                }
            }
        }

        stage('Push Image to Docker Hub') {
            environment {
                registryCredential = 'dockerhublogin'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', registryCredential) {
                        dockerImage.push("latest")
                    }
                }
            }
        }

        stage('Push Image to AWS ECR') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
                                     string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY')]) {
                        sh "aws ecr get-login-password --region ${awsRegion} | docker login --username AWS --password-stdin ${awsAccountId}.dkr.ecr.${awsRegion}.amazonaws.com"
                        dockerImage.tag("${awsAccountId}.dkr.ecr.${awsRegion}.amazonaws.com/${ecrRepository}:latest")
                        docker.withRegistry("${awsAccountId}.dkr.ecr.${awsRegion}.amazonaws.com", 'ecr:latest') {
                            dockerImage.push("latest")
                        }
                    }
                }
            }
        }

        stage('Deploy App to Kubernetes') {
            steps {
                script {
                    kubernetesDeploy(configs: "deploymentservice.yml", kubeconfigId: "kubernetes")
                }
            }
        }
    }
}
