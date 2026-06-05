pipeline {
    agent any

    stages {

        stage('Get Code') {
            steps {
                sh '''
                    whoami
                    hostname
                    curl -L -o samconfig.toml https://raw.githubusercontent.com/adabaja/todo-list-aws-config/staging/samconfig.toml
                    cat samconfig.toml
                '''
            }
        }

        stage('Static Test') {
            steps {
                sh '''
                    flake8 --exit-zero --format=pylint src > flake8.out
                    bandit -r src -f txt -o bandit.out || true
                '''
                archiveArtifacts artifacts: 'flake8.out,bandit.out', allowEmptyArchive: true
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    sam build
                    sam deploy --config-env staging --no-confirm-changeset --no-fail-on-empty-changeset --region us-east-1
                '''
            }
        }

        stage('Rest Test') {
            steps {
                sh '''
                    export BASE_URL=$(aws cloudformation describe-stacks --stack-name todo-list-aws-staging --region us-east-1 --query "Stacks[0].Outputs[?OutputKey=='BaseUrlApi'].OutputValue" --output text)
                    echo "BASE_URL=$BASE_URL"
                    python3 -m pytest --junitxml=result-rest.xml test/integration/todoApiTest.py
                '''
                junit 'result-rest.xml'
            }
        }
    }
}
