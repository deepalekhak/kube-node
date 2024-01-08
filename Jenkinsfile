pipeline {
    environment {
        AWS_DEFAULT_REGION = "us-east-1"  // Replace with your actual AWS region
        AWS_ACCOUNT_ID = "866762610186"   // Replace with your actual AWS account ID
        IMAGE_REPO_NAME = "kube-node"     // Replace with your actual ECR repository name
        IMAGE_TAG = "v1"              // Replace with your desired image tag
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
        dockerImageName = "thetips4you/nodeapp"
    }

    agent any

    stages {
        stage('Checkout Source') {
            steps {
                sh 'echo passed'
                //git branch: 'main', url: 'https://github.com/deepalekhak/kube-node.git'
            }
        }

        stage('Build Image') {
            steps {
                script {
                    dockerImage = docker.build dockerImageName
                }
            }
        }

        stage('Logging into AWS ECR') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
                                     string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY')]) {
                        sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
                    }
                }       
            }
        }

        stage('Pushing to ECR') {
            steps {
                script {
                    def ecrTaggedImage = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
                    sh "docker build -t $ecrTaggedImage ."
                    sh "docker push $ecrTaggedImage"
                }
            }
        }

        stage('Deploy App to K8s') {
            steps {
                script {
                 sudo sh ' kubectl apply -f deployment.yml'
                }
            }
        }
    }
}
