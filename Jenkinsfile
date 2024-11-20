pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "kommuarjun/spring-petclinic"  // Replace with your Docker image name
        DOCKER_CREDENTIALS_ID = "docker-hub-credentials-id"  // Jenkins Docker Hub credentials ID
        DEPLOYMENT_FILE = "k8s/deployment.yaml"  // Path to your deployment file
        SERVICE_FILE = "k8s/service.yaml"  // Path to your service file
        EKS_NAMESPACE = "default"  // Replace with your desired namespace
        IMAGE_TAG = "${BUILD_NUMBER}"  // Use Jenkins build number as image tag
    }
    stages {
        stage('Clone Repository') {
            steps {
                echo "Cloning the repository..."
                git branch: 'main', url: 'https://github.com/KommuArjun/spring-petclinic.git'
            }
        }
        stage('Build Application') {
            steps {
                echo "Building the application..."
                sh './mvnw clean package -DskipTests'
            }
        }
        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh """
                    docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} .
                """
            }
        }
        stage('Push to Docker Hub') {
            steps {
                echo "Pushing Docker image to Docker Hub..."
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin
                        docker push ${DOCKER_IMAGE}:${IMAGE_TAG}
                    """
                }
            }
        }
        stage('Update Kubernetes Manifest') {
            steps {
                echo "Updating Kubernetes deployment file with the new Docker image..."
                sh """
                    export DOCKER_IMAGE=${DOCKER_IMAGE}
                    export IMAGE_TAG=${IMAGE_TAG}
                    # Substitute environment variables into deployment.yaml
                    envsubst < ${DEPLOYMENT_FILE} > updated-deployment.yaml
                    cat updated-deployment.yaml  # Optional: To check if the substitution worked
                """
            }
        }
        stage('Deploy to EKS') {
            agent {
                label 'eks'  // Specify the Jenkins node labeled as 'eks'
            }
            steps {
                echo "Deploying to EKS..."
                sh """
                    kubectl apply -f ${SERVICE_FILE} -n ${EKS_NAMESPACE}
                    kubectl apply -f updated-deployment.yaml -n ${EKS_NAMESPACE}
                """
            }
        }
    }
    post {
        success {
            echo "Pipeline executed successfully!"
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}
