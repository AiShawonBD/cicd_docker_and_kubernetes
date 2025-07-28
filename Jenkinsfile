pipeline {
    agent any

    environment {
        IMAGE_NAME = 'aishawon/cicd_docker_demo'
        IMAGE_TAG = "${IMAGE_NAME}:${env.GIT_COMMIT}"
        // Remove AWS and KUBECONFIG credentials for local k8s
    }

    stages {
        stage('Setup') {
            steps {
                // No kubeconfig file needed, local kubectl uses default config
                sh "pip install -r requirements.txt"
            }
        }

        stage('Test') {
            steps {
                sh "pytest"
            }
        }

        stage('Login to docker hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh 'echo ${PASSWORD} | docker login -u ${USERNAME} --password-stdin'
                }
                echo 'Login successfully'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_TAG} ."
                echo "Docker image built successfully"
                sh "docker image ls"
            }
        }

        stage('Push Docker Image') {
            steps {
                sh "docker push ${IMAGE_TAG}"
                echo "Docker image pushed successfully"
            }
        }

        stage('Deploy Pod') {
            steps {
                script {
                    // Dynamically replace image tag in your deployment YAML before applying
                    sh """
                       sed -i 's|image: aishawon/cicd_docker_demo:[a-f0-9]*|image: ${IMAGE_TAG}|g' k8s/deployment.yaml
                    """
                    // Apply deployment to local Kubernetes cluster
                    sh "kubectl apply -f k8s/deployment.yaml"
                }
            }
        }

        stage('Acceptance Test') {
            steps {
                script {
                    // For local k8s, usually no LoadBalancer IP, use ClusterIP or NodePort
                    def service = sh(script: "kubectl get svc flask-app-service -o jsonpath='{.spec.clusterIP}:{.spec.ports[0].port}'", returnStdout: true).trim()
                    echo "Service endpoint: ${service}"

                    sh "k6 run -e SERVICE=${service} acceptance-test.js"
                }
            }
        }
    }
}
