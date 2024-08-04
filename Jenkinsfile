pipeline {
    agent any
    
    stages{
        stage("Code"){
            steps{
                git url: "https://github.com/anubhavdwivedy/two-tier-app.git", branch: "jenkins"
            }
        }
        stage("Build & Test"){
            steps{
                    sh "docker push anubhavdwivedy/flaskapp:latest" 
                }
            }
        }
        stage("Deploy"){
            steps{
                sh "docker-compose down && docker-compose up -d"
            }
        }
    }
