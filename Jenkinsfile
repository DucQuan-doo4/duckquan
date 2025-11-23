pipeline {
    agent any
    environment {
        AWS_REGION = "us-east-1"
        ECR_REPO = "591313757404.dkr.ecr.us-east-1.amazonaws.com/exam2"
        CLUSTER_NAME = "exam2"
        SERVICE_NAME = "exam2-service-s1aceznj"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/DucQuan-doo4/duckquan.git'
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
                sh """
                docker tag exam2:${IMAGE_TAG} ${ECR_REPO}:${IMAGE_TAG}
                docker push ${ECR_REPO}:${IMAGE_TAG}
                """
            }
        }

        stage('Register Task Definition') {
            steps {
                sh """
                # Tạo file task-definition.json tạm thời
                cat > task-definition.json <<EOF
                {
                  "family": "${SERVICE_NAME}",
                  "containerDefinitions": [
                    {
                      "name": "container1",
                      "image": "${ECR_REPO}:${IMAGE_TAG}",
                      "memory": 512,
                      "cpu": 256,
                      "essential": true,
                      "portMappings": [
                        {
                          "containerPort": 3000,
                          "hostPort": 3000
                        }
                      ]
                    }
                  ]
                }
                EOF

                aws ecs register-task-definition --cli-input-json file://task-definition.json
                """
            }
        }

        stage('Deploy ECS') {
            steps {
                sh """
                # Lấy task definition ARN mới nhất
                TASK_DEF_ARN=\$(aws ecs list-task-definitions --family-prefix ${SERVICE_NAME} --sort DESC --max-items 1 | jq -r '.taskDefinitionArns[0]')
                
                # Update service
                aws ecs update-service --cluster ${CLUSTER_NAME} --service ${SERVICE_NAME} --force-new-deployment --task-definition \$TASK_DEF_ARN
                """
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
