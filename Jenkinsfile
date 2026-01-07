pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "mrudulsmohan/zomato-react:latest"
        AWS_REGION  = "ap-south-1"
        EKS_CLUSTER = "zomato-cluster"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE} .'
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                }
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                sh 'docker push ${DOCKER_IMAGE}'
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                     credentialsId: 'aws-creds']
                ]) {
                    sh '''
                    set -e

                    echo "===== AWS IDENTITY ====="
                    aws sts get-caller-identity

                    echo "===== UPDATE KUBECONFIG ====="
                    aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER}

                    echo "===== VERIFY CLUSTER ACCESS ====="
                    kubectl get nodes

                    echo "===== DEPLOY APPLICATION ====="
                    kubectl apply -f k8s/deployment.yaml
                    '''
                }
            }
        }
    }
}
