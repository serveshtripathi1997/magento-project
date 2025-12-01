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

        stage('Start Docker Services') {
            steps {
                sh 'docker compose up -d'
            }
        }

        stage('Composer Auth Setup') {
            steps {
                sh '''
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
                '''
            }
        }

        stage('Install Magento') {
            steps {
                sh '''
                docker run --rm \
                    -v $PWD/src:/var/www/html \
                    -v $PWD/src/auth/auth.json:/composer/auth.json \
                    composer:2.8 \
                    composer create-project \
                    --repository-url=https://repo.magento.com/ \
                    magento/project-community-edition=2.4.8-p3 .
                '''
            }
        }

    }
}
