pipeline {
    agent any
    tools {
        nodejs 'nodejs'
    }

    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
       stage ('Gitclone') {
            steps {
                git 'https://github.com/vjpatil176/hotstarproject.git'
            }
        }
        stage ('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'hotstar-token') {
                         sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=Hotstarpro -Dsonar.projectName=Hotstarpro "
                    }
                }
            }
        }
        stage ('S3 backup') {
            steps {
                sh 'aws s3 cp /var/lib/jenkins/workspace/hotstar-project/ s3://hotstar-backup/backup-for-pro/ --recursive'
            }
        }
        stage ('deploy to tomcat') {
            steps {
                 sh "sudo scp -o StrictHostKeyChecking=no /var/lib/jenkins/workspace/hotstar-project/package.json package-lock.json root@44.196.59.129:/root/tomcat/webapps/"
            }
        }
        stage ('Build Docker Image') {
            steps {
                withDockerRegistry(credentialsId: 'docker-token', tool name: 'docker') {
                   sh "docker build -t vjpatil176/hotstar-project:v1 ." 
                }
            }
        }
        stage ('Push Docker Image to DckerHub') {
            steps {
                withDockerRegistry(credentialsId: 'docker-token', tool name: 'docker') {
                   sh "docker push vjpatil176/hotstar-project:v1" 
                }
            }
        }
        stage ('deploy to Docker container') {
            steps {
                withDockerRegistry(credentialsId: 'docker-token', tool name: 'docker') {
                   sh "docker run -d -p 3000:3000 vjpatil176/hotstar-project:v1" 
                }
            }
        }
    }
}
