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
