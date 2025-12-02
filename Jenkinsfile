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
                dir("${WORKSPACE}/magento-project") {

                    echo "===== STOPPING OLD SERVICES ====="
                    sh "docker compose down || true"

                    echo "===== BUILDING PHP IMAGE (NO CACHE) ====="
                    sh "docker compose build --no-cache php"

                    echo "===== STARTING DOCKER SERVICES ====="
                    sh "docker compose up -d"
                }
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
                            --ad
