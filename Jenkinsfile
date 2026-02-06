pipeline {
    agent any

    environment {
        KEY_PATH = "/var/lib/jenkins/jenkinskkp.pem"
    }

    stages {

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
                }
            }
        }

        stage('Deploy Website') {
            steps {
                sh """
                scp -r -i ${KEY_PATH} -o StrictHostKeyChecking=no website/* ubuntu@${SERVER_IP}:/tmp/
                """

                sh """
                ssh -i ${KEY_PATH} -o StrictHostKeyChecking=no ubuntu@${SERVER_IP} "
                sudo rm -rf /var/www/html/* &&
                sudo mv /tmp/* /var/www/html/ &&
                sudo systemctl restart nginx"
                """
            }
        }
    }
}
