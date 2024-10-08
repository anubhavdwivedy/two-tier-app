pipeline {
    agent any

    stages {  
         stage('Checkout') {
            steps {
                // Checkout code from the main branch
                git branch: 'main', url: 'https://github.com/anubhavdwivedy/two-tier-app.git'
            }
        }
         
        stage('Create_infra') {
            steps {
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws-jenkins', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')])  { // save your credentials in jenkins crdentials then get the syntax with pipeline syntax
                    dir('terraform') {
                         sh 'terraform init'
                         sh 'terraform plan -out=tfplan'
                         sh 'terraform apply -auto-approve tfplan'
                    }
                }
            }
        }
    
        stage('Extract required IP & Port') {
            steps {
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws-jenkins', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')])  {
                    dir('terraform') {
                        script {
                            // Extract the backend private IP from Terraform output
                     
                            env.BACKEND_PUBLIC_IP = sh(script: "terraform output -raw backend_public_ip", returnStdout: true).trim()
                            env.BACKEND_PRIVATE_IP = sh(script: "terraform output -raw backend_private_ip", returnStdout: true).trim()
                            env.FRONTEND_PUBLIC_IP = sh(script: "terraform output -raw frontend_public_ip", returnStdout: true).trim()
                            env.FRONTEND_PORT = sh(script: "terraform output -raw frontend_port", returnStdout: true).trim()
                        }
                    }  
                }  
            }
        }
    
        stage('Update and Deploy Applications') {
            steps {
                sshagent(['devopstest']) {// save your private key credentials in ssh agent throuh plugins 
                    script {
                        sh  """
                        backend_ip=${env.BACKEND_PRIVATE_IP}
                            """

                        sh  """
                            ssh -o StrictHostKeyChecking=no -i /home/anubhav/2-tier-app-key.pem ubuntu@${env.BACKEND_PUBLIC_IP} 'chmod +x /home/ubuntu/backend.sh && ./backend.sh' //use the private key path of yours 
                            ssh -o StrictHostKeyChecking=no -i /home/anubhav/2-tier-app-key.pem ubuntu@${env.BACKEND_PUBLIC_IP} 'sudo docker pull  mysql:5.7'
                            ssh -o StrictHostKeyChecking=no -i /home/anubhav/2-tier-app-key.pem ubuntu@${env.BACKEND_PUBLIC_IP} 'sudo docker run -d -p 3306:3306 mysql:5.7'
                            """
                        sh  """
                            ssh -o StrictHostKeyChecking=no -i /home/anubhav/2-tier-app-key.pem ubuntu@${env.FRONTEND_PUBLIC_IP} 'chmod +x /home/ubuntu/frontend.sh && ./frontend.sh'
                            ssh -o StrictHostKeyChecking=no -i /home/anubhav/2-tier-app-key.pem ubuntu@${env.FRONTEND_PUBLIC_IP} 'sudo docker pull anubhavdwivedy/flaskapp:latest'
                            ssh -o StrictHostKeyChecking=no -i /home/anubhav/2-tier-app-key.pem ubuntu@${env.FRONTEND_PUBLIC_IP} 'sudo docker run -d -p 5000:5000 -e DB_HOST=10.0.2.48 anubhavdwivedy/flaskapp:latest'//in db_host we have to use the private ip of the backend
                            """
                    }
                }  
            } 
        }     
        stage('Test_solution') {
            steps {
                sshagent(['devopstest']) {
                    script{
                        echo "frontend Public IP: ${env.FRONTEND_PUBLIC_IP}"
                        def frontend_address = "http://${env.FRONTEND_PUBLIC_IP}:${env.FRONTEND_PORT}"
                        echo "Frontend Application URL: ${frontend_address}"
                        // Assign the URL to an environment variable
                        env.FRONTEND_URL = frontend_address
                    }
                }
            }  
        }
    } 
    post {
        always{
            echo "Pipeline completed"
            echo "Frontend URL: ${env.FRONTEND_URL}"
        }
    }
}
