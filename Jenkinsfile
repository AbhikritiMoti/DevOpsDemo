pipeline {
    agent { label 'docker' }

    triggers {
        pollSCM('H/2 * * * *')
    }

    environment {
        IMAGE_NAME   = 'pythonapp'
        TAG          = "${BUILD_NUMBER}"
        LOCAL_IMAGE  = "${IMAGE_NAME}:${TAG}"

        ECR_REGISTRY = '820242921378.dkr.ecr.us-east-1.amazonaws.com'
        ECR_REPO     = 'pythonapp'
        FULL_IMAGE   = "${ECR_REGISTRY}/${ECR_REPO}:${TAG}"

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

        stage('Show Build Info') {
            steps {
                sh '''
                    echo "BUILD_NUMBER=${BUILD_NUMBER}"
                    echo "LOCAL_IMAGE=${LOCAL_IMAGE}"
                    echo "FULL_IMAGE=${FULL_IMAGE}"
                '''
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
                sh '''
                    sudo docker build -t ${LOCAL_IMAGE} .
                    sudo docker tag ${LOCAL_IMAGE} ${FULL_IMAGE}
                '''
            }
        }

        stage('Docker Container Creation') {
            steps {
                sh '''
                    export IMAGE_NAME=${LOCAL_IMAGE}
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
                    docker push ${FULL_IMAGE}
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
                    pwd
                    ls -la
                    ls -la k8s

                    test -f k8s/configmap.yaml
                    test -f k8s/redis-deploy.yaml
                    test -f k8s/redis-service.yaml
                    test -f k8s/py-deploy.yaml
                    test -f k8s/py-service.yaml

                    sed "s|__IMAGE__|${FULL_IMAGE}|g" k8s/py-deploy.yaml | kubectl apply -f -
                    kubectl apply -f k8s/configmap.yaml
                    kubectl apply -f k8s/redis-deploy.yaml
                    kubectl apply -f k8s/redis-service.yaml
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
