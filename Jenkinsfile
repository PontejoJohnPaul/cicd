pipeline {
    agent any

    environment {
        GIT_REPO_URL = 'https://github.com/PontejoJohnPaul/cicd.git'
        GIT_CREDENTIALS_ID = 'github-pat'
        GIT_BRANCH = 'main'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "*/${env.GIT_BRANCH}"]],
                    userRemoteConfigs: [[
                        url: "${env.GIT_REPO_URL}",
                        credentialsId: "${env.GIT_CREDENTIALS_ID}"
                    ]]
                ])
            }
        }

        stage('Detect Change') {
            steps {
                script {
                    def changed = sh(
                        script: "git diff --name-only HEAD~1 HEAD | grep '.php' | head -n 1",
                        returnStdout: true
                    ).trim()

                    env.TARGET_PHP_FILE = changed ?: "index.php"
                    echo "Changed file: ${env.TARGET_PHP_FILE}"
                }
            }
        }

        stage('Stage & Debug Mode') {
            steps {
                sh '''
                echo "Staging files..."

                sudo mkdir -p /var/www/html/staging

                sudo rsync -av --delete \
                --exclude='venv/' \
                --exclude='.git/' \
                ./ /var/www/html/staging/

                echo "php_flag display_errors On" | sudo tee /var/www/html/staging/.htaccess
                echo "php_value error_reporting 32767" | sudo tee -a /var/www/html/staging/.htaccess

                sudo chown -R www-data:www-data /var/www/html/staging
                '''
            }
        }

        stage('Run Test') {
            steps {
                sh '''
                echo "Running tests..."

                python3 -m venv venv
                . venv/bin/activate

                pip install selenium

                # Optional test file
                if [ -f test.py ]; then
                    python3 test.py
                else
                    echo "No test.py found, skipping test"
                fi
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                echo "Deploying to production..."

                sudo rsync -av --delete \
                --exclude='venv/' \
                --exclude='.git/' \
                --exclude='staging/' \
                ./ /var/www/html/

                sudo chown -R www-data:www-data /var/www/html/

                echo "Deployment complete!"
                '''
            }
        }
    }
}
