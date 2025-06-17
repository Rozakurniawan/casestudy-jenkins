pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
    }

    stages {
        stage('Checkout Source') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t texsa/demo-app:latest .'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh '''
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                            docker push texsa/demo-app:latest
                        '''
                    }
                }
            }
        }

        stage('Deploy to Kubernetes (Helm)') {
            agent {
                docker {
                    image 'alpine/helm:3.14.0'
                    args '--entrypoint=""'
                }
            }
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-dev', variable: 'KUBE_FILE')]) {
                    script {
                        sh '''
                            export KUBECONFIG=$KUBE_FILE
                            helm upgrade --install casestudy-jenkins1 ./helm \
                              --set image.repository=texsa/demo-app \
                              --set image.tag=latest \
                              --namespace default --create-namespace
                        '''
                    }
                }
            }
        }
    }

    post {
        failure {
            echo "❌ Pipeline Gagal: Cek log untuk mengetahui error"
        }
        success {
            echo "✅ Pipeline Berhasil: Aplikasi sudah dideploy ke Kubernetes"
        }
    }
}
