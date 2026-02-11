pipeline {
    agent any

    environment {
        KEY_PATH = "/var/lib/jenkins/mak_1.pem"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/NiranjanaYuvaraj08/ci-cd-game'
            }
        }

        stage('Terraform Init') {
            steps {
                dir('terraform') {
                    sh 'terraform init'
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                dir('terraform') {
                    sh 'terraform apply -auto-approve'
                }
            }
        }

        stage('Get Server IP') {
            steps {
                dir('terraform') {
                    script {
                        env.SERVER_IP = sh(
                            script: "terraform output -raw public_ip",
                            returnStdout: true
                        ).trim()
                    }
                    echo "Server IP is ${SERVER_IP}"
                }
            }
        }

        stage('Wait for Server to be Ready') {
            steps {
                echo "Waiting for EC2 instance to finish booting..."
                sh 'sleep 60'
            }
        }

        stage('Deploy Website') {
    steps {
        withCredentials([sshUserPrivateKey(credentialsId: 'ec2-key',
                                           keyFileVariable: 'SSH_KEY')]) {
            sh """
            scp -r -i $SSH_KEY -o StrictHostKeyChecking=no website/* ubuntu@${SERVER_IP}:/tmp/
            """
        }
    }
}


    post {
        success {
            echo "🎉 Deployment Successful! Visit: http://${SERVER_IP}"
        }
        failure {
            echo "❌ Pipeline Failed. Check logs above."
        }
    }
}
