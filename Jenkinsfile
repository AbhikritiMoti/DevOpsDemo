pipeline {
    agent { labels 'docker' }
    triggers {
        pollSCM('H/2 * * * *')   // every 2 minutes
    }

    stages {
        stage('Clone Repository') {
            steps {
git branch: 'main',
                    url: 'git@github.com:ManjuSKM/devOpsonAWSTraining.git',
                    credentialsId: 'github-creds'                
            }
        }

    stage('Check the docker cli') {
      steps {
        sh "docker --version"
        sh "docker ps -a"
      }
    }
        stage('Docker-Build') {
            steps {
                echo "Docker-Build step here..."
                sh "docker build . -t pythonapp:v1"
            }
        }

        stage('DockerContainer Creation') {
            steps {
                echo "Running Docker compose here..."
                sh "docker compose up -d"
            }
        }

        stage('Docker-Build') {
            steps {
                echo "Docker-Build step here..."
                sh "docker build . -t pythonapp:v1"
            }
        }
         stage('Image push') {
            steps {
                echo "Docker-Build step here..."
                sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 985539761450.dkr.ecr.us-east-1.amazonaws.com"
                sh "docker tag pythonapp:v1 985539761450.dkr.ecr.us-east-1.amazonaws.com/pythonapp:v1"
                sh "docker push 985539761450.dkr.ecr.us-east-1.amazonaws.com/pythonapp:v1"
        }
         }
    }
}
