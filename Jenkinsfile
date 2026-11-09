pipeline {
    agent { label 'docker-agent' }

    environment {
        DOCKER_HUB_USER = 'hallcash'
        BACKEND_IMAGE_NAME = 'crudback'
        DATABASE_IMAGE_NAME = 'mysql'
        GIT_REPO = 'https://github.com/hallcash/crudapp.git'
        MANAGER_IP = '192.168.0.1'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                git branch: 'main', url: "${GIT_REPO}"
            }
        }

        stage('Build Docker Images') {
            steps {
                sh 'docker build -f php.Dockerfile -t ${DOCKER_HUB_USER}/${BACKEND_IMAGE_NAME}:${BUILD_NUMBER} .'
                sh 'docker build -f mysql.Dockerfile -t ${DOCKER_HUB_USER}/${DATABASE_IMAGE_NAME}:${BUILD_NUMBER} .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                    sh 'docker push ${DOCKER_HUB_USER}/${BACKEND_IMAGE_NAME}:${BUILD_NUMBER}'
                    sh 'docker push ${DOCKER_HUB_USER}/${DATABASE_IMAGE_NAME}:${BUILD_NUMBER}'
                }
            }
        }

        stage('Deploy to Swarm') {
            steps {
                sh 'docker stack deploy -c docker-compose.yaml app'
            }
        }
    }
}
