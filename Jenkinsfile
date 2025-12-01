pipeline {
    agent any

    environment {
        MAGENTO_AUTH = credentials('magento-marketplace-auth')
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Docker Cleanup') {
            steps {
                sh '''
                echo "===== CLEANING UP DOCKER ENVIRONMENT ====="
                docker stop $(docker ps -aq) || true
                docker rm -f $(docker ps -aq) || true
                docker network prune -f || true
                docker volume prune -f || true
                docker image prune -af || true
                echo "===== CLEANUP COMPLETE ====="
                '''
            }
        }

        stage('Start Docker Services') {
            steps {
                sh '''
                echo "===== STARTING DOCKER SERVICES ====="
                docker compose up -d
                echo "===== DOCKER SERVICES RUNNING ====="
                '''
            }
        }

        stage('Composer Auth Setup') {
            steps {
                sh """
                echo "===== GENERATING COMPOSER AUTH FILE ====="
                mkdir -p src/auth

                # Write JSON WITHOUT expanding Jenkins variables
                cat > src/auth/auth.json << 'EOF'
{
  "http-basic": {
    "repo.magento.com": {
      "username": "1610fb57f5cf7efc5737918d19252904",
      "password": "ec7a19daf6a93ffbf817d25423e545f1"
    }
  }
}
EOF

                # Safely insert Jenkins credentials
                sed -i "s/__USERNAME__/${MAGENTO_AUTH_USR}/" src/auth/auth.json
                sed -i "s/__PASSWORD__/${MAGENTO_AUTH_PSW}/" src/auth/auth.json

                chmod 600 src/auth/auth.json
                echo "===== AUTH FILE CREATED ====="
                """
            }
        }

        stage('Install Magento Code (Composer)') {
            steps {
                sh '''
                echo "===== INSTALLING MAGENTO VIA COMPOSER ====="

                docker run --rm \
                    -e COMPOSER_HOME=/composer \
                    -v $PWD/src:/var/www/html \
                    -v $PWD/src/auth:/composer \
                    composer:2.8 \
                    composer create-project \
                        --repository-url=https://repo.magento.com/ \
                        magento/project-community-edition=2.4.8-p3 .

                echo "===== MAGENTO CODE INSTALLED ====="
                '''
            }
        }

        stage('Install Magento System (PHP Container)') {
            steps {
                sh '''
                echo "===== RUNNING MAGENTO SETUP:INSTALL ====="

                docker exec magento-php php bin/magento setup:install \
                    --base-url=http://localhost:8090 \
                    --db-host=mariadb \
                    --db-name=magento \
                    --db-user=root \
                    --db-password=root123 \
                    --backend-frontname=admin \
                    --search-engine=opensearch \
                    --opensearch-host=opensearch \
                    --opensearch-port=9200 \
                    --amqp-host=rabbitmq \
                    --amqp-user=magento \
                    --amqp-password=magento123

                echo "===== MAGENTO INSTALLED SUCCESSFULLY ====="
                '''
            }
        }
    }

    post {
        success {
            echo "🎉 Magento Deployment Completed Successfully!"
            echo "Frontend:  http://localhost:8090"
            echo "Admin URL: http://localhost:8090/admin"
        }
        failure {
            echo "❌ Build Failed — check Jenkins console output."
        }
    }
}

