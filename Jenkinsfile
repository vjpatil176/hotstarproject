pipeline {
    agent any
    
    tools {
        nodejs 'nodejs'
    }
    
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('GitClone from GitHub') {
            steps {
                git 'https://github.com/vjpatil176/hotstarproject.git'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=Hotstarpro -Dsonar.projectName=Hotstarpro "
                }
            }
        }
        
        stage('S3 Backup') {
            steps {
                sh 'aws s3 cp /var/lib/jenkins/workspace/hotstarpro/ s3://hotstarpro/hotstarpro/ --recursive'
            }
        }
        
        stage('Deploy to Tomcat') {
            steps {
                sh "sudo scp -o StrictHostKeyChecking=no /var/lib/jenkins/workspace/hotstarpro/package.json package-lock.json root@34.207.242.242:/root/tomcat/webapps/"
            }
        }
        
        stage('Docker Image Build&Tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-token', toolName: 'docker') {
                        sh "docker build -t adikesavanaidug2404/hotstarpro:v1 ."
                    }
                }
            }
        }
        
        stage('Push Docker Image to DockerHub') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-token', toolName: 'docker') {
                        sh "docker push adikesavanaidug2404/hotstarpro:v1"
                    }
                }
            }
        }
        
        stage('Deploy to DockerContainer(Run the DockerImage') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-token', toolName: 'docker') {
                        sh "docker run -d -p 3000:3000 adikesavanaidug2404/hotstarpro:v1 ."
                    }
                }
            }
        }
    }
}

