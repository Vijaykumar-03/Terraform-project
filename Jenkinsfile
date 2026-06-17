pipeline {

    agent any

    environment {

        AWS_REGION = 'ap-south-1'
        ACCOUNT_ID = '732231074251'

        ECR_REPO = 'devops-app'

        IMAGE_TAG = "${BUILD_NUMBER}"

        EKS_CLUSTER = 'devops-eks'

        DEPLOYMENT_NAME = 'devops-app'
        CONTAINER_NAME  = 'devops-app'
    }

    stages {

        stage('Clean Workspace') {

            steps {
                cleanWs()
            }
        }

        stage('Clone Repository') {

            steps {

                git branch: 'main',
                url: 'https://github.com/Vijaykumar-03/Terraform-project.git'
            }
        }

        stage('Build Docker Image') {

            steps {

                sh '''
                docker build -t ${ECR_REPO}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Login To ECR') {

            steps {

                sh '''
                aws ecr get-login-password \
                --region ${AWS_REGION} | \
                docker login \
                --username AWS \
                --password-stdin \
                ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                '''
            }
        }

        stage('Tag Docker Image') {

            steps {

                sh '''
                docker tag ${ECR_REPO}:${IMAGE_TAG} \
                ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}

                docker tag ${ECR_REPO}:${IMAGE_TAG} \
                ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:latest
                '''
            }
        }

        stage('Push To ECR') {

            steps {

                sh '''
                docker push ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}

                docker push ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:latest
                '''
            }
        }

        stage('Configure EKS') {

            steps {

                sh '''
                aws eks update-kubeconfig \
                --region ${AWS_REGION} \
                --name ${EKS_CLUSTER}
                '''
            }
        }

        stage('Deploy To EKS') {

            steps {

                sh '''
                kubectl apply -f deployment.yaml

                kubectl apply -f service.yaml
                '''
            }
        }

        stage('Update Deployment') {

            steps {

                sh '''
                kubectl set image deployment/${DEPLOYMENT_NAME} \
                ${CONTAINER_NAME}=${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}

                kubectl rollout status deployment/${DEPLOYMENT_NAME}
                '''
            }
        }

        stage('Verify Deployment') {

            steps {

                sh '''
                kubectl get pods
                kubectl get deployment
                kubectl get svc
                '''
            }
        }
    }

    post {

        success {

            echo 'Application Successfully Deployed to EKS'
        }

        failure {

            echo 'Deployment Failed'
        }
    }
}
