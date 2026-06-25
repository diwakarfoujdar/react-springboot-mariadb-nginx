```groovy
pipeline {

    agent any

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main',
                url: 'https://github.com/diwakarfoujdar/react-springboot-mariadb-nginx.git'
            }
        }

        stage('Build Backend Docker Image') {
            steps {
                dir('backend') {
                    sh 'docker build -t student-backend:v2 .'
                }
            }
        }

        stage('Build Frontend Docker Image') {
            steps {
                dir('frontend') {
                    sh 'docker build -t student-frontend:v1 .'
                }
            }
        }

        stage('Build Nginx Docker Image') {
            steps {
                dir('nginx') {
                    sh 'docker build -t student-nginx:v1 .'
                }
            }
        }

        stage('Push Images To DockerHub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {

                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin

                    docker tag student-backend:v2 diwakar2010/student-backend:v2
                    docker push diwakar2010/student-backend:v2

                    docker tag student-frontend:v1 diwakar2010/student-frontend:v1
                    docker push diwakar2010/student-frontend:v1

                    docker tag student-nginx:v1 diwakar2010/student-nginx:v1
                    docker push diwakar2010/student-nginx:v1

                    docker logout
                    '''
                }
            }
        }

        stage('Deploy Application') {
            steps {
                sh '''

                echo "Stopping old containers..."
                docker stop nginx frontend backend mariadb || true

                echo "Removing old containers..."
                docker rm nginx frontend backend mariadb || true

                echo "Creating Docker network..."
                docker network create student-network || true

                echo "Pulling latest images..."
                docker pull mariadb:11
                docker pull diwakar2010/student-backend:v2
                docker pull diwakar2010/student-frontend:v1
                docker pull diwakar2010/student-nginx:v1

                echo "Starting MariaDB..."
                docker run -d \
                  --name mariadb \
                  --network student-network \
                  -v mariadb-data:/var/lib/mysql \
                  -e MARIADB_ROOT_PASSWORD=root1234 \
                  -e MARIADB_DATABASE=student_db \
                  -e MARIADB_USER=diwakar \
                  -e MARIADB_PASSWORD=linux7107 \
                  mariadb:11

                echo "Waiting for MariaDB..."
                sleep 20

                echo "Starting Backend..."
                docker run -d \
                  --name backend \
                  --network student-network \
                  -p 8081:8080 \
                  -e DB_URL=jdbc:mariadb://mariadb:3306/student_db \
                  -e DB_USER=diwakar \
                  -e DB_PASSWORD=linux7107 \
                  diwakar2010/student-backend:v2

                echo "Starting Frontend..."
                docker run -d \
                  --name frontend \
                  --network student-network \
                  diwakar2010/student-frontend:v1

                echo "Starting Nginx..."
                docker run -d \
                  --name nginx \
                  --network student-network \
                  -p 80:80 \
                  diwakar2010/student-nginx:v1

                '''
            }
        }
    }

    post {

        success {
            echo 'CI/CD Pipeline executed successfully.'
        }

        failure {
            echo 'CI/CD Pipeline failed.'
        }

        always {
            echo 'Pipeline execution completed.'
        }
    }
}
```
