pipeline {
    agent any

    environment {
        IMAGE_NAME = 'aishawon/cicd_docker_demo'
        IMAGE_TAG = "${IMAGE_NAME}:${GIT_COMMIT}"
    }

    stages {
        stage('Setup') {
            steps {
                sh 'pip install -r requirements.txt'
            }
        }

        stage('Test') {
            steps {
                sh 'pytest'
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh 'echo $PASSWORD | docker login -u $USERNAME --password-stdin'
                }
                echo 'Docker login successful'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_TAG .'
                sh 'docker image ls'
                echo 'Docker image built successfully'
            }
        }

        stage('Push Docker Image') {
            steps {
                sh 'docker push $IMAGE_TAG'
                echo 'Docker image pushed successfully'
            }
        }

        stage('Deploy to Local Kubernetes') {
            steps {
                script {
                    sh """
                        sed -i 's|image: aishawon/cicd_docker_demo:[a-zA-Z0-9_.-]*|image: $IMAGE_TAG|g' k8s/deployment.yaml
                    """
                    sh 'kubectl apply -f k8s/' //Deploy all k8s file.
                }
            }
        }

        stage('Acceptance Test') {
            steps {
                script {
                    sleep(time: 10, unit: 'SECONDS')
                    def nodePort = sh(script: "kubectl get svc flask-app-service -o jsonpath='{.spec.ports[0].nodePort}'", returnStdout: true).trim()
                    def nodeIP = sh(script: "minikube ip", returnStdout: true).trim()
                    def serviceURL = "http://${nodeIP}:${nodePort}"
                    echo "Running acceptance test on ${serviceURL}"
                    sh "k6 run -e SERVICE=${serviceURL} acceptance-test.js"
                }
            }
        }
    }
}
