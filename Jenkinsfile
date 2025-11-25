pipeline {
    agent any

    environment {
        DOCKER_HOST = "tcp://host.docker.internal:2375"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/Samuelw7876/evaluacion3-seguridad.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "Docker host: $DOCKER_HOST"
                    docker version
                    docker build -t evaluacion3-app ./app
                '''
            }
        }

        stage('Run App') {
            steps {
                sh '''
                    docker run -d --name app-test -p 5000:5000 evaluacion3-app
                    sleep 6
                '''
            }
        }

        stage('Basic Test') {
            steps {
                sh 'curl -f http://localhost:5000'
            }
        }
    }

    post {
        always {
            sh '''
                docker stop app-test || true
                docker rm app-test || true
            '''
        }
    }
}
