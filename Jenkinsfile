pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "kommuarjun/spring-petclinic"  // Docker image name
        DOCKER_CREDENTIALS_ID = "docker-hub-credentials-id"  // Docker Hub credentials ID in Jenkins
        EKS_NAMESPACE = "default"  // Kubernetes namespace
        IMAGE_TAG = "${BUILD_NUMBER}"  // Using Jenkins build number as the image tag
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
        stage('Deploy to EKS') {
            agent {
                label 'eks'  // Use Jenkins node with kubectl installed
            }
            steps {
                echo "Deploying to EKS..."

                // Imperative kubectl commands to deploy the app and create the service
                sh """
                    # Imperative kubectl command to create the deployment
                    kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-petclinic
  namespace: ${EKS_NAMESPACE}
spec:
  replicas: 2
  selector:
    matchLabels:
      app: spring-petclinic
  template:
    metadata:
      labels:
        app: spring-petclinic
    spec:
      containers:
      - name: spring-petclinic
        image: ${DOCKER_IMAGE}:${IMAGE_TAG}
        ports:
        - containerPort: 8080
EOF

                    # Imperative kubectl command to create the service (NodePort)
                    kubectl apply -f - <<EOF
apiVersion: v1
kind: Service
metadata:
  name: spring-petclinic
  namespace: ${EKS_NAMESPACE}
spec:
  selector:
    app: spring-petclinic
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: NodePort
EOF
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
