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

        stage('Promote') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-token', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                    sh '''
                        git config user.email "jenkins@unir.com"
                        git config user.name "Jenkins CI"
                        git config merge.ours.driver true
                        git checkout -- samconfig.toml
                        git fetch origin
                        git checkout -B master origin/master
                        git merge origin/develop --no-edit
                        git push https://${GIT_USER}:${GIT_TOKEN}@github.com/adabaja/unir_cp1.4.git master
                    '''
                }
            }
        }

    }
}
