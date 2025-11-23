pipeline {
    agent any
    environment {
        AWS_REGION = "us-east-1"
        ECR_REPO = "591313757404.dkr.ecr.us-east-1.amazonaws.com/exam2"
        CLUSTER_NAME = "exam2"
        SERVICE_NAME = "exam2-service-s1aceznj"
        CONTAINER_NAME = "container1"
        IMAGE_TAG = ""   // khai báo trước để dùng toàn pipeline
        TASK_DEF_ARN = "" // khai báo luôn
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/DucQuan-doo4/duckquan.git'
            }
        }

        stage('Get Commit Hash') {
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
                    sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}"
                }
            }
        }

        stage('Tag & Push') {
            steps {
                sh "docker tag exam2:${IMAGE_TAG} ${ECR_REPO}:${IMAGE_TAG}"
                sh "docker push ${ECR_REPO}:${IMAGE_TAG}"
            }
        }

        stage('Register ECS Task Definition') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    script {
                        def taskDefFile = "ecs-task-def-${IMAGE_TAG}.json"
                        sh "sed 's|<IMAGE_TAG>|${IMAGE_TAG}|g' ecs-task-def-template.json > ${taskDefFile}"
                        TASK_DEF_ARN = sh(script: "aws ecs register-task-definition --cli-input-json file://${taskDefFile} --query taskDefinition.taskDefinitionArn --output text", returnStdout: true).trim()
                        echo "TASK_DEF_ARN: ${TASK_DEF_ARN}"
                    }
                }
            }
        }

        stage('Deploy ECS') {
            steps {
                script {
                    sh """
                    aws ecs update-service \
                        --cluster ${CLUSTER_NAME} \
                        --service ${SERVICE_NAME} \
                        --task-definition ${TASK_DEF_ARN} \
                        --force-new-deployment
                    """
                }
            }
        }
    }

    post {
        success {
            echo "CI/CD pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed! Check logs."
        }
    }
}
