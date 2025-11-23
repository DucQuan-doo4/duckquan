pipeline {
    agent any
    environment {
        AWS_REGION = "us-east-1"
        ECR_REPO = "591313757404.dkr.ecr.us-east-1.amazonaws.com/exam2"
        CLUSTER_NAME = "exam2"
        SERVICE_NAME = "exam2-service-s1aceznj"
        CONTAINER_NAME = "container1"
        IMAGE_TAG = "latest"
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/DucQuan-doo4/duckquan.git'
            }
        }

        stage('Build Docker') {
            steps {
                sh 'docker build -t exam2 .'
            }
        }

        stage('Login to ECR') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh '''
                    aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO
                    '''
                }
            }
        }

        stage('Tag & Push') {
            steps {
                sh '''
                docker tag exam2:latest $ECR_REPO:latest
                docker push $ECR_REPO:latest
                '''
            }
        }

        stage('Deploy ECS') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh '''
                    aws ecs update-service \
                        --cluster $CLUSTER_NAME \
                        --service $SERVICE_NAME \
                        --force-new-deployment
                    '''
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
