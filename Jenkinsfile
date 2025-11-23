pipeline {
    agent any
    environment {
        AWS_REGION = "us-east-1"
        ECR_REPO = "591313757404.dkr.ecr.us-east-1.amazonaws.com/exam2"
        CLUSTER_NAME = "exam2"
        SERVICE_NAME = "exam2-service-s1aceznj"
        CONTAINER_NAME = "container1"
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/DucQuan-doo4/duckquan.git'
            }
        }

        stage('Set Image Tag') {
            steps {
                script {
                    IMAGE_TAG = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    echo "Using image tag: ${IMAGE_TAG}"
                }
            }
        }

        stage('Build Docker') {
            steps {
                sh "docker build -t exam2:${IMAGE_TAG} ."
            }
        }

        stage('Login to ECR') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh "aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO"
                }
            }
        }

        stage('Push Docker') {
            steps {
                sh """
                docker tag exam2:${IMAGE_TAG} $ECR_REPO:${IMAGE_TAG}
                docker push $ECR_REPO:${IMAGE_TAG}
                """
            }
        }

        stage('Register New Task Definition') {
            steps {
                script {
                    // Lấy cấu hình task hiện tại
                    def taskDefJson = sh(script: "aws ecs describe-task-definition --task-definition $SERVICE_NAME", returnStdout: true).trim()
                    
                    // Lấy container definition hiện tại
                    def containerDef = sh(script: "echo '$taskDefJson' | jq -c '.taskDefinition.containerDefinitions'", returnStdout: true).trim()
                    
                    // Tạo task definition mới với image mới
                    sh """
                    aws ecs register-task-definition \\
                        --family $SERVICE_NAME \\
                        --network-mode awsvpc \\
                        --requires-compatibilities FARGATE \\
                        --cpu 256 --memory 512 \\
                        --execution-role-arn arn:aws:iam::591313757404:role/ecsTaskExecutionRole \\
                        --container-definitions '[{ "name": "$CONTAINER_NAME", "image": "$ECR_REPO:${IMAGE_TAG}", "essential": true, "portMappings": [{"containerPort":80,"hostPort":80}] }]'
                    """
                }
            }
        }

        stage('Deploy ECS Service') {
            steps {
                sh """
                LATEST_REV=$(aws ecs list-task-definitions --family-prefix $SERVICE_NAME --sort DESC --max-items 1 | jq -r '.taskDefinitionArns[0]')
                aws ecs update-service --cluster $CLUSTER_NAME --service $SERVICE_NAME --task-definition $LATEST_REV
                """
            }
        }
    }

    post {
        success { echo "CI/CD pipeline completed successfully!" }
        failure { echo "Pipeline failed! Check logs." }
    }
}
