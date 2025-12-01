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

                echo "Stopping all running containers (ignore errors)..."
                docker stop $(docker ps -aq) || true

                echo "Removing all containers (ignore errors)..."
                docker rm -f $(docker ps -aq) || true

                echo "Removing orphaned networks..."
                docker network prune -f || true

                echo "Removing dangling volumes..."
                docker volume prune -f || true

                echo "Removing unused images..."
                docker image prune -af || true

                echo "===== DOCKER CLEANUP COMPLETE ====="
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
                sh '''
                echo "===== GENERATING COMPOSER AUTH FILE ====="
                mkdir -p src/auth

                cat <<EOF > src/auth/auth.json
                {
                  "http-basic": {
                    "repo.magento.com": {
                      "username": "${MAGENTO_AUTH_USR}",
                      "password": "${MAGENTO_AUTH_PSW}"
                    }
                  }
                }
                EOF

                echo "===== AUTH FILE GENERATED ====="
                '''
            }
        }

        stage('Install Magento Code (Composer)') {
            steps {
                sh '''
                echo "===== INSTALLING MAGENTO VIA COMPOSER ====="

                docker run --rm \
                    -v $PWD/src:/var/www/html \
                    -v $PWD/src/auth/auth.json:/composer/auth.json \
                    composer:2.8 \
                    composer create-project \
                    --repository-url=https://repo.magento.com/ \
                    magento/project-community-edition=2.4.8-p3 .

                echo "===== MAGENTO COMPOSER INSTALL COMPLETE ====="
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

                echo "===== MAGENTO INSTALLATION COMPLETE ====="
                '''
            }
        }

    }
}
