pipeline {
    agent any

    environment {
        PROJECT_DIR = "/home/serveshtripathi/magento-project"
        MAGENTO_DIR = "/var/www/html"
    }

    stages {

        stage('Checkout Source') {
            steps {
                echo "===== CHECKING OUT SOURCE CODE ====="
                checkout scm
            }
        }

        stage('Start Docker Services') {
            steps {
                echo "===== STARTING DOCKER SERVICES ====="
                dir("${PROJECT_DIR}") {
                    sh "docker compose down || true"
                    sh "docker compose up -d"
                }
                sh "sleep 10"
            }
        }

        stage('Clean Web Directory') {
            steps {
                echo "===== CLEANING /var/www/html DIRECTORY ====="

                sh """
                    docker exec magento-php bash -lc '
                        rm -rf ${MAGENTO_DIR}/*
                    '
                """
            }
        }

        stage('Install Magento Source Code') {
            steps {
                echo "===== INSTALLING MAGENTO SOURCE USING COMPOSER CONFIG ====="

                withCredentials([usernamePassword(credentialsId: 'magento-repo-creds',
                                                  usernameVariable: 'MAGENTO_AUTH_USR',
                                                  passwordVariable: 'MAGENTO_AUTH_PSW')]) {

                    sh """
                    docker exec magento-php bash -lc '
                        composer config --global http-basic.repo.magento.com ${MAGENTO_AUTH_USR} ${MAGENTO_AUTH_PSW}
                        composer create-project \
                            --repository-url=https://repo.magento.com/ \
                            magento/project-community-edition=2.4.8-p3 \
                            ${MAGENTO_DIR}
                    '
                    """
                }
            }
        }

        stage('Magento Setup Install') {
            steps {
                echo "===== RUNNING MAGENTO SETUP:INSTALL ====="

                sh """
                    docker exec magento-php bash -lc '
                        php ${MAGENTO_DIR}/bin/magento setup:install \
                            --base-url="http://localhost:8090/" \
                            --db-host="magento-db" \
                            --db-name="magento" \
                            --db-user="root" \
                            --db-password="root123" \
                            --admin-firstname="Servesh" \
                            --admin-lastname="Tripathi" \
                            --admin-email="admin@example.com" \
                            --admin-user="admin" \
                            --admin-password="Admin@1234" \
                            --backend-frontname="adminpanel" \
                            --search-engine="opensearch" \
                            --opensearch-host="magento-opensearch" \
                            --opensearch-port=9200 \
                            --amqp-host="magento-rabbitmq" \
                            --amqp-port=5672 \
                            --amqp-user="magento" \
                            --amqp-password="magento123" \
                            --cache-backend="redis" \
                            --cache-backend-redis-server="magento-valkey" \
                            --cache-backend-redis-port=6379 \
                            --page-cache-redis-server="magento-valkey" \
                            --page-cache-redis-port=6379 \
                            --session-save=redis \
                            --session-save-redis-host="magento-valkey" \
                            --session-save-redis-port=6379 \
                            --use-rewrites=1
                    '
                """
            }
        }

        stage('Post-Install Setup') {
            steps {
                echo "===== FINALIZING MAGENTO ENVIRONMENT ====="

                sh """
                    docker exec magento-php bash -lc '
                        php ${MAGENTO_DIR}/bin/magento deploy:mode:set developer
                        php ${MAGENTO_DIR}/bin/magento cache:flush
                        chmod -R 777 ${MAGENTO_DIR}
                    '
                """
            }
        }
    }

    post {
        success {
            echo "🎉 Magento Build + Install Completed Successfully!"
        }
        failure {
            echo "❌ Build Failed — check Jenkins console output."
        }
    }
}
