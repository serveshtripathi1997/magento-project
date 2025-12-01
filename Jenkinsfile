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

                chmod 600 src/auth/auth.json
                '''
            }
        }

stage('Composer Auth Setup') {
    steps {
        sh '''
        echo "===== GENERATING COMPOSER AUTH FILE ====="
        mkdir -p src/auth

        cat > src/auth/auth.json << 'EOF'
{
  "http-basic": {
    "repo.magento.com": {
      "username": "'"${MAGENTO_AUTH_USR}"'",
      "password": "'"${MAGENTO_AUTH_PSW}"'"
    }
  }
}
EOF

        chmod 600 src/auth/auth.json
        echo "===== AUTH FILE GENERATED ====="
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

