pipeline {
    agent any

    environment {
        IMAGE_NAME = 'aishawon/cicd_docker_demo'
        IMAGE_TAG = "${IMAGE_NAME}:${env.GIT_COMMIT}"
    }

    stages {
        stage('Install Dependencies') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    . venv/bin/activate
                    pytest
                '''
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh 'echo ${PASSWORD} | docker login -u ${USERNAME} --password-stdin'
                }
                echo 'Login successful'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_TAG} .'
                sh 'docker image ls'
            }
        }

        stage('Push Docker Image') {
            steps {
                sh 'docker push ${IMAGE_TAG}'
            }
        }

        stage('Deploy to Local K8s') {
            steps {
                script {
                    sh "sed -i 's|image: .*|image: ${IMAGE_TAG}|' k8s/deployment.yaml"
                }
                sh 'kubectl apply -f k8s/'
                sh 'kubectl get all'
            }
        }

        stage('Run Acceptance Test') {
            steps {
                script {
                    sh "sleep 10"
                    def podName = sh(script: "kubectl get pods -l app=flask-app -o jsonpath='{.items[0].metadata.name}'", returnStdout: true).trim()
                    sh "kubectl logs ${podName}"
                }
            }
        }
    }
}
