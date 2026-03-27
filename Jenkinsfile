pipeline {
    agent { label 'docker' }

    triggers {
        pollSCM('H/2 * * * *')
    }

    environment {
        IMAGE_NAME   = 'pythonapp:v1'
        ECR_REGISTRY = '820242921378.dkr.ecr.us-east-1.amazonaws.com'
        ECR_REPO     = '820242921378.dkr.ecr.us-east-1.amazonaws.com/pythonapp:v1'
        AWS_REGION   = 'us-east-1'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main',
                    url: 'git@github.com:AbhikritiMoti/DevOpsDemo.git',
                    credentialsId: 'github-ssh'
            }
        }

        stage('Check Docker CLI') {
            steps {
                sh '''
                    sudo docker --version
                    sudo docker ps -a || true
                    docker-compose --version
                '''
            }
        }

        stage('Cleanup old containers') {
            steps {
                sh '''
                    docker-compose down || true
                    sudo docker rm -f git-pp02_web_1 git-pp02_redis_1 git-pp022_web_1 git-pp022_redis_1 || true
                '''
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Docker build running'
                sh '''
                    sudo docker build -t ${IMAGE_NAME} .
                '''
            }
        }

        stage('Docker Container Creation') {
            steps {
                echo 'Running docker-compose here...'
                sh '''
                    docker-compose up -d
                '''
            }
        }

        stage('Checking Output') {
            steps {
                sh '''
                    sudo docker ps
                    docker-compose ps
                '''
            }
        }

        stage('App Working') {
            steps {
                sh '''
                    curl localhost:9010
                '''
            }
        }

        stage('Image Push') {
            steps {
                sh '''
                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}
                    docker tag ${IMAGE_NAME} ${ECR_REPO}
                    docker push ${ECR_REPO}
                '''
            }
        }

        stage('Check kube env') {
            steps {
                sh '''
                    minikube status
                    kubectl get nodes
                '''
            }
        }

        stage('Deploy kube files') {
            steps {
                sh '''
                    echo "Current workspace:"
                    pwd
                    echo "Workspace files:"
                    ls -la
                    echo "k8s folder contents:"
                    ls -la k8s

                    test -f k8s/configmap.yaml
                    test -f k8s/redis-deploy.yaml
                    test -f k8s/redis-service.yaml
                    test -f k8s/py-deploy.yaml
                    test -f k8s/py-service.yaml

                    kubectl apply -f k8s/configmap.yaml
                    kubectl apply -f k8s/redis-deploy.yaml
                    kubectl apply -f k8s/redis-service.yaml
                    kubectl apply -f k8s/py-deploy.yaml
                    kubectl apply -f k8s/py-service.yaml
                '''
            }
        }

        stage('App testing on kube env') {
            steps {
                sh '''
                    kubectl get pods -o wide
                    kubectl get svc
                    curl 192.168.49.2:30115
                '''
            }
        }
    }

    post {
        always {
            sh '''
                echo "===== Docker Containers ====="
                sudo docker ps -a || true

                echo "===== Kubernetes Pods ====="
                kubectl get pods -o wide || true

                echo "===== Kubernetes Services ====="
                kubectl get svc || true
            '''
        }
        success {
            echo 'Pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed. Check the logs above for the exact stage and error.'
        }
    }
}
