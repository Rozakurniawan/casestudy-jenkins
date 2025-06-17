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
                    echo "🛠 Building image texsa/demo-app:latest..."
                    sh 'docker build -t texsa/demo-app:latest .'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    echo "📦 Pushing image to DockerHub..."
                    sh """
                        echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin
                        docker push texsa/demo-app:latest
                    """
                }
            }
        }

        stage('Deploy to Kubernetes (Helm)') {
            agent {
                docker {
                    image 'alpine/helm:3.14.0'
                }
            }
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-dev', variable: 'KUBE_FILE')]) {
                    script {
                        echo "🚀 Deploying to Kubernetes via Helm..."
                        sh """
                            export KUBECONFIG=$KUBE_FILE
                            helm upgrade --install casestudy-jenkins1 ./helm \
                              --set image.repository=texsa/demo-app \
                              --set image.tag=latest \
                              --namespace default --create-namespace
                        """
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
