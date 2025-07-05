pipeline {
    agent any

    environment {
        GIT_REPO = 'https://github.com/fdelgadodyvenia/todo-list-aws.git'
        BRANCH = 'develop'
        AWS_REGION = 'us-east-1'
        STACK_NAME = 'todo-stack'
        S3_BUCKET = 'aws-sam-cli-managed-default-samclisourcebucket-hgwhprjqmlgt'
    }

    stages {

        stage('📥 Get Code') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'd579908c-4755-490c-af17-6888c97748fd', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                    sh """
                        rm -rf todo-list-aws
                        echo "🔽 Cloning repository"
                        git clone --branch \$BRANCH https://\$GIT_USER:\$GIT_PASS@github.com/fdelgadodyvenia/todo-list-aws.git
                    """
                }
            }
        }

        stage('🧹 StaticTests') {
            steps {
                dir('todo-list-aws/src') {
                    sh """
                        python3 -m venv venv
                        . venv/bin/activate
    
                        echo "📎 Installing tools"
                        pip install flake8 flake8-html bandit

                        echo "🧪 Running Flake8"
                         flake8 . --exit-zero --format=html --htmldir=flake8_html

                        echo "🔒 Running Bandit"
                        bandit -r . -f html -o bandit_report.html || true
                    """
                }
            }

        post {
                always {
                    echo "📂 Archiving flake8 and bandit reports"
                    archiveArtifacts artifacts: 'todo-list-aws/src/flake8_html/**, todo-list-aws/src/bandit_report.html', allowEmptyArchive: true
            
                    publishHTML([
                        reportDir: 'todo-list-aws/src/flake8_html',
                        reportFiles: 'index.html',
                        reportName: 'Flake8 Report',
                        allowMissing: false,
                        alwaysLinkToLastBuild: true,
                        keepAll: true
                    ])
            
                    publishHTML([
                        reportDir: 'todo-list-aws/src',
                        reportFiles: 'bandit_report.html',
                        reportName: 'Bandit Report',
                        allowMissing: false,
                        alwaysLinkToLastBuild: true,
                        keepAll: true
                    ])
                }
            }
        }

        stage('🚀 SAM Deploy') {
            steps {
                dir('todo-list-aws') {
                    sh """
                        echo "🔧 Building SAM Application"
                        sam build

                        echo "🚀 Deploying to Staging"
                        sam deploy \
                            --stack-name $STACK_NAME \
                            --region $AWS_REGION \
                            --s3-bucket $S3_BUCKET \
                            --capabilities CAPABILITY_IAM \
                            --no-confirm-changeset \
                            --no-fail-on-empty-changeset
                    """
                }
            }
        }

        stage('🧪 RestTest') {
            steps {
                dir('todo-list-aws') {
                    withEnv(["BASE_URL=https://y3789gn804.execute-api.us-east-1.amazonaws.com/Prod"]){
                        sh """
                            echo "🐍 Setting up virtual environment for integration tests"
                            python3 -m venv venv
                            . venv/bin/activate
                            pip install --upgrade pip
                            pip install -r requirements.txt || pip install pytest requests
    
                            echo "✅ Running pytest for integration tests"
                            pytest test/integration/todoApiTest.py --exitfirst --disable-warnings
                        """
                    }
                }
            }
        }

        stage('📦 Promote') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'd579908c-4755-490c-af17-6888c97748fd', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                    sh """
                        echo "📤 Promoting code to main"
                        cd todo-list-aws
                        git config user.name "jenkins"
                        git config user.email "jenkins@example.com"
                        git checkout main
                        git merge \$BRANCH --no-edit
                        git push https://\$GIT_USER:\$GIT_PASS@github.com/fdelgadodyvenia/todo-list-aws.git main
                    """
                }
            }
        }
    }
}
