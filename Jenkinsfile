pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1' 
        ECR_REPO = 'circleci' 
        ECR_REGISTRY = '244706281787.dkr.ecr.ap-south-1.amazonaws.com/circleci' 
        IMAGE_TAG = "${env.BUILD_ID}" 
        DOCKER_IMAGE = "${ECR_REGISTRY}:${IMAGE_TAG}" 
    }

    tools {
        nodejs 'NodeJS_16' 
    }

    stages {
        stage('Clone Repo') {
            steps {
                echo 'Cloning repo...'
                git branch: 'main', url: 'https://github.com/AnkitSingh9496/JekinsPipeline.git' // Your GitHub URL
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Installing dependencies...'
                sh 'npm install'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('Login to ECR') {
            steps {
                echo 'Logging in to Amazon ECR...'
                sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}"
            }
        }

        stage('Push Image to ECR') {
            steps {
                echo 'Pushing Docker image to ECR...'
                sh "docker push ${DOCKER_IMAGE}"
            }
        }

        stage('Deploy to EKS') {
            steps {
                echo 'Deploying to EKS...'
                script {
                    
                    sh "sed -i 's|image:.*|image: ${DOCKER_IMAGE}|' k8s/deployment.yaml"
                    
                    sh "kubectl apply -f k8s/"
                }
            }
        }
    }
}

// pipeline {
//     agent any

//     tools {
//         nodejs 'NodeJS_16' 
//     }

//     stages {
//         stage('Clone Repo') {
//             steps {
//                 echo 'Cloning repo...'
//                 git branch: 'main', url: 'https://github.com/AnkitSingh9496/JekinsPipeline.git'
//             }
//         }

//         stage('Install Dependencies') {
//             steps {
//                 echo 'Installing dependencies...'
//                 sh 'npm install'
//             }
//         }

//         stage('Run App') {
//             steps {
//                 echo 'Starting app...'
//                 sh 'node index.js &'
//                 sh 'sleep 5'
//             }
//         }
//     }
// }
