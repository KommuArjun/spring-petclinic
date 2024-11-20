pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "your-dockerhub-username/spring-petclinic" // Replace with your Docker Hub username and image name
        DOCKER_CREDENTIALS_ID = "docker-hub-credentials-id"      // Replace with your Jenkins Docker Hub credentials ID
        DEPLOYMENT_FILE = "k8s/deployment.yaml"                   // Path to your deployment file in the repository
        SERVICE_FILE = "k8s/service.yaml"                         // Path to your service file in the repository
        EKS_NAMESPACE = "default"                                 // Replace with your namespace
        IMAGE_TAG = "${BUILD_NUMBER}"                             // Use Jenkins build number as the image tag
    }
    stages {
        stage('Clone Repository') {
            steps {
                echo "Cloning the repository..."
                git 'https://github.com/KommuArjun/spring-petclinic.git'
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
                    sed -i 's|image: .*|image: ${DOCKER_IMAGE}:${IMAGE_TAG}|' ${DEPLOYMENT_FILE}
                """
            }
        }
        stage('Deploy to EKS') {
            agent {
                label 'eks' 
            }
            steps {
                echo "Deploying to EKS..."
                sh """
                    kubectl apply -f ${SERVICE_FILE} -n ${EKS_NAMESPACE}
                    kubectl apply -f ${DEPLOYMENT_FILE} -n ${EKS_NAMESPACE}
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
