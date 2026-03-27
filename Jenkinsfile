pipeline {
    agent { label 'docker' }
    triggers {
        pollSCM('H/2 * * * *')   // every 2 minutes
    }
 
    stages {
        stage('Clone Repository') {
            steps {
                    git branch: 'main',
                    url: 'git@github.com:ManjuSKM/devOpsonAWSTraining.git',
                    credentialsId: 'github-ssh'
            }
        }
 
        stage('Check the docker cli') {
            steps {
                sh "sudo docker --version"
                sh "sudo docker ps -a"
                sh "docker-compose --version"
            }
        }
        stage('Docker build') {
            steps {
                echo "Docker build running"
                sh "sudo docker build . -t pythonapp:v1"
            }
        }
 
        stage('Docker Container Cerateion') {
            steps {
                echo "Running docker-compose here..."
                sh "docker-compose up -d"
            }
        }
 
        stage('Cheking output') {
            steps {
               sh "sudo docker ps"
                sh "docker-compose ps"
            }
        }
        // stage('Hosting Nginx FrontEnd') {
        //     steps {
        //        sh "sudo yum install nginx -y"
        //        sh "sudo systemctl start nginx" 
        //     }
        // }
 
        stage('app working') {
            steps {
               sh "curl localhost:9010"
            }
        }
        stage('Image push') {
            steps {
               sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 820242921378.dkr.ecr.us-east-1.amazonaws.com"
               sh "docker tag pythonapp:v1 820242921378.dkr.ecr.us-east-1.amazonaws.com/pythonapp:v1"
                sh "docker push 820242921378.dkr.ecr.us-east-1.amazonaws.com/pythonapp:v1"
            }
        }
 
     stage('Check kube env') {
            steps {
                sh """
                minikube status
                kubectl get nodes
                """
            }
        }
        stage('deploy kube files') {
            steps {
                sh """
                kubectl apply -f py-redis-configmap
                """
            }
        }
 
        stage('App testing on kube env') {
            steps {
               sh "curl 192.168.49.2:30115"
            }
        }
    }
}
