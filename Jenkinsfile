pipeline {
    agent any

    parameters {
        string(name: 'APP_TAG', defaultValue: 'v1.0.0', description: 'Docker image tag to deploy')
    }

    stages {

        stage('Get Code') {
            steps {
                checkout scm
            }
        }

        stage('Provision EC2') {
            steps {
                withCredentials([
                    string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY'),
                    string(credentialsId: 'aws-session-token', variable: 'AWS_SESSION_TOKEN')
                ]) {
                    sh '''
                    export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                    export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                    export AWS_SESSION_TOKEN=$AWS_SESSION_TOKEN

                    cd ansible_files
                    ansible-playbook provision_prod.yml
                    '''
                }
            }
        }

        stage('Deploy App') {
            steps {
                sh """
                cd ansible_files
                ansible-playbook deploy_app.yml --extra-vars "app_tag=${params.APP_TAG}" --private-key ~/devops-lab-project/ansible_files/flask_running_app_staging_key.pem -u ubuntu
                """
            }
        }

        stage('Verify App') {
            steps {
                withCredentials([
                    string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY'),
                    string(credentialsId: 'aws-session-token', variable: 'AWS_SESSION_TOKEN')
                ]) {
                    sh '''
                    export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                    export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                    export AWS_SESSION_TOKEN=$AWS_SESSION_TOKEN

                    PROD_IP=$(aws ec2 describe-instances \
                      --filters "Name=tag:Name,Values=devops-prod" "Name=instance-state-name,Values=running" \
                      --query "Reservations[0].Instances[0].PublicIpAddress" \
                      --output text)

                    curl http://$PROD_IP/
                    '''
                }
            }
        }
    }
}
