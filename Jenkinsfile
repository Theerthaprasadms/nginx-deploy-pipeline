pipeline {
    agent any

    parameters {
        choice(name: 'ENVIRONMENT', choices: ['dev', 'prod'], description: 'Deployment Environment')
    }

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO = "774305596656.dkr.ecr.ap-south-1.amazonaws.com/nginx-${params.ENVIRONMENT}"
        IMAGE_TAG = "${params.ENVIRONMENT}-${env.BUILD_NUMBER}"
    }

    stages {
        stage('Clone') {
            steps {
                git url: 'https://github.com/Theerthaprasadms/nginx-deploy-pipeline.git'
            }
        }

        stage('Test Dockerfile') {
            steps {
                sh '''
                docker build -t nginx-test .
                docker run -d -p 8080:80 --name nginx-test nginx-test
                sleep 5
                STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:8080)
                if [ "$STATUS" != "200" ]; then
                  echo "Health check failed"
                  exit 1
                fi
                docker stop nginx-test
                docker rm nginx-test
                '''
            }
        }

        stage('Build and Push to ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-cred']]) {
                    sh '''
                    aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO
                    docker build -t $ECR_REPO:$IMAGE_TAG .
                    docker push $ECR_REPO:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                docker pull $ECR_REPO:$IMAGE_TAG
                docker stop nginx-container || true
                docker rm nginx-container || true
                docker run -d -p 8081:80 --name nginx-container $ECR_REPO:$IMAGE_TAG
                '''
            }
        }
    }

    post {
        success {
            echo "Visit NGINX at: http://13.201.19.40:8081"
        }
    }
}
