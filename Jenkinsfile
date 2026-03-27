pipeline {
    agent { label 'docker' }

    triggers {
        pollSCM('H/2 * * * *')
    }

    environment {
        IMAGE_NAME   = 'pythonapp:v1'
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
                sh 'sudo docker --version'
                sh 'sudo docker ps -a || true'
                sh 'docker-compose --version'
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
                sh 'sudo docker build -t $IMAGE_NAME .'
            }
        }

        stage('Docker Container Creation') {
            steps {
                echo 'Running docker-compose here...'
                sh 'docker-compose up -d'
            }
        }

        stage('Checking Output') {
            steps {
                sh 'sudo docker ps'
                sh 'docker-compose ps'
            }
        }

        stage('App Working') {
            steps {
                sh 'curl localhost:9010'
            }
        }

        stage('Image Push') {
            steps {
                sh 'aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin 820242921378.dkr.ecr.us-east-1.amazonaws.com'
                sh 'docker tag $IMAGE_NAME $ECR_REPO'
                sh 'docker push $ECR_REPO'
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
                    ls -la py-redis-configmap || true

                    kubectl apply -f py-redis-configmap/configmap.yaml
                    kubectl apply -f redis-deploy.yaml
                    kubectl apply -f redis-service.yaml
                    kubectl apply -f py-deploy.yaml
                    kubectl apply -f py-service.yaml
                '''
            }
        }

        stage('App testing on kube env') {
            steps {
                sh 'curl 192.168.49.2:30115'
            }
        }
    }

    post {
        always {
            sh 'sudo docker ps -a || true'
            sh 'kubectl get pods -o wide || true'
            sh 'kubectl get svc || true'
        }
    }
}
