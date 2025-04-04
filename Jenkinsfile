pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'   // Change to your AWS region
        AWS_ACCOUNT_ID = '703671891941'  // Change to your AWS Account ID
        REPO_NAME = 'springboot-repo' // Change to your ECR repository name
        IMAGE_TAG = "latest"
        ECR_REPO_URL = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${REPO_NAME}"
        ECS_CLUSTER = 'springboot-cluster'
        ECS_SERVICE = 'springboot-service'
        ECS_TASK_DEF = 'springboot-task'
    }

    stages {
        
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/venakteshmuddada/spring-boot-app.git'
            }
        }

        stage('Build JAR with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Login to AWS ECR') {
            steps {
                sh 'aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO_URL'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $ECR_REPO_URL:$IMAGE_TAG .'
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                sh 'docker push $ECR_REPO_URL:$IMAGE_TAG'
            }
        }

        stage('Update ECS Task Definition') {
            steps {
                sh '''
                TASK_DEF_ARN=$(aws ecs describe-task-definition --task-definition $ECS_TASK_DEF --query 'taskDefinition.taskDefinitionArn' --output text)
                aws ecs register-task-definition --cli-input-json "$(aws ecs describe-task-definition --task-definition $ECS_TASK_DEF --query 'taskDefinition' | jq '.taskDefinitionArn=null')"
                '''
            }
        }

        stage('Deploy to ECS') {
            steps {
                sh 'aws ecs update-service --cluster $ECS_CLUSTER --service $ECS_SERVICE --force-new-deployment'
            }
        }
    }
}
