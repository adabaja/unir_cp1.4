pipeline {
    agent any

    stages {

        stage('Get Code') {
            steps {
                sh '''
                    whoami
                    hostname
                    curl -L -o samconfig.toml https://raw.githubusercontent.com/adabaja/todo-list-aws-config/production/samconfig.toml
                    cat samconfig.toml
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    sam build
                    sam deploy --config-env production --no-confirm-changeset --no-fail-on-empty-changeset --region us-east-1
                '''
            }
        }

        stage('Rest Test') {
            steps {
                sh '''
                    export BASE_URL=$(aws cloudformation describe-stacks --stack-name todo-list-aws-production --region us-east-1 --query "Stacks[0].Outputs[?OutputKey=='BaseUrlApi'].OutputValue" --output text)
                    echo "BASE_URL=$BASE_URL"
                    python3 -m pytest -m readonly --junitxml=result-rest.xml test/integration/todoApiTest.py
                '''
                junit 'result-rest.xml'
            }
        }
    }
}
