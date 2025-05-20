
pipeline {
    agent any

    environment {
        AWS_REGION   = 'ap-south-1'
        ECR_REPO     = 'circleci'
        ECR_REGISTRY = "244706281787.dkr.ecr.${AWS_REGION}.amazonaws.com"
        IMAGE_TAG    = "${BUILD_ID}"
        DOCKER_IMAGE = "${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG}"
    }

    triggers {
        pollSCM('* * * * *') // Enables GitHub webhook trigger for pipeline
    }

    tools {
        nodejs 'NodeJS_16'
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    echo 'Cloning GitHub repository...'
                    sh 'rm -rf JekinsPipeline'
                    sh 'git clone https://github.com/AnkitSingh9496/JekinsPipeline.git'
                }
            }
        }

        stage('Install Node.js Dependencies') {
            steps {
                dir('JekinsPipeline') {
                    echo 'Installing Node.js packages...'
                    sh '''
                        export PATH=$PATH:$NODEJS_HOME/bin
                        npm install
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('JekinsPipeline') {
                    echo "Building Docker image: ${DOCKER_IMAGE}"
                    sh "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }

        stage('Authenticate with AWS ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-jenkins-creds' // <-- Replace this with your actual credential ID
                ]]) {
                    echo 'Authenticating with ECR...'
                    sh """
                        aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}
                    """
                }
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                echo "Pushing Docker image: ${DOCKER_IMAGE}"
                sh "docker push ${DOCKER_IMAGE}"
            }
        }

        stage('Deploy to Amazon EKS') {
            steps {
                dir('JekinsPipeline') {
                    echo 'Deploying to EKS...'
                    withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: 'aws-jenkins-creds'
                    ]]) {
                        sh """
                            aws eks update-kubeconfig --region ${AWS_REGION} --name CircleCI
                            sed -i 's|image:.*|image: ${DOCKER_IMAGE}|' k8s/deployment.yaml
                            kubectl apply -f k8s/deployment.yaml
                            kubectl apply -f k8s/service.yaml
                        """
                    }
                }
            }
        }
    }
}


























// pipeline {
//     agent any

//     environment {
//         AWS_REGION            = 'ap-south-1'
//         ECR_REPO              = 'circleci'
//         ECR_REGISTRY          = "244706281787.dkr.ecr.${AWS_REGION}.amazonaws.com"
//         IMAGE_TAG             = "${BUILD_ID}"
//         DOCKER_IMAGE          = "${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG}"
//         // Replace it with your actual AWS Credentials
//         AWS_ACCESS_KEY_ID     = ''
//         AWS_SECRET_ACCESS_KEY = ''
//         AWS_SESSION_TOKEN     = ''
//     }

//     tools {
//         nodejs 'NodeJS_16'
//     }

//     stages {
//         stage('Clone Repository') {
//             steps {
//                 script {
//                     echo 'Cleaning previous repo clone if exists...'
//                     sh 'rm -rf JekinsPipeline'

//                     echo 'Cloning repository manually...'
//                     sh 'git clone https://github.com/AnkitSingh9496/JekinsPipeline.git'
//                 }
//             }
//         }

//         stage('Install Node.js Dependencies') {
//             steps {
//                 dir('JekinsPipeline') {
//                     echo 'Installing Node.js dependencies...'
//                     sh '''
//                         export PATH=$PATH:$NODEJS_HOME/bin
//                         npm install
//                     '''
//                 }
//             }
//         }

//         stage('Build Docker Image') {
//             steps {
//                 dir('JekinsPipeline') {
//                     echo "Building Docker image: ${DOCKER_IMAGE}"
//                     sh "docker build -t ${DOCKER_IMAGE} ."
//                 }
//             }
//         }

//         stage('Authenticate with ECR') {
//             steps {
//                 echo 'Authenticating with AWS ECR...'
//                 script {
//                     sh """
//                         aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}
//                     """
//                 }
//             }
//         }

//         stage('Push Docker Image to ECR') {
//             steps {
//                 echo "Pushing Docker image: ${DOCKER_IMAGE}"
//                 sh "docker push ${DOCKER_IMAGE}"
//             }
//         }

//         stage('Deploy to Amazon EKS') {
//             steps {
//                 dir('JekinsPipeline') {
//                     echo 'Deploying the application to EKS...'
//                     script {
//                         // Set up kubeconfig
//                         sh """
//                             aws eks update-kubeconfig --region ${AWS_REGION} --name CircleCI
//                         """

//                         echo "Listing files in the directory..."
//                         sh "ls -la k8s"

//                         // Modify the deployment.yaml file and apply it
//                         sh """
//                             sed -i 's|image:.*|image: ${DOCKER_IMAGE}|' k8s/deployment.yaml
//                             kubectl apply -f k8s/deployment.yaml
//                             kubectl apply -f k8s/service.yaml
//                         """
//                     }
//                 }
//             }
//         }
//     }
// }
