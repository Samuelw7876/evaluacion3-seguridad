pipeline {
    agent any

    environment {
        DOCKER_HOST = "tcp://host.docker.internal:2375"
    }

    stages {
        stage('Checkout') {
            steps {
                git url: "https://github.com/Samuelw7876/evaluacion3-seguridad.git", branch: "master"
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Docker host: ${DOCKER_HOST}"
                sh 'docker version'
                sh 'docker build -t evaluacion3-app ./app'
            }
        }

        stage('Run App') {
            steps {
                sh 'docker run -d --name app-test -p 5000:5000 evaluacion3-app'
                sh 'sleep 6'
            }
        }

        stage('Basic Test') {
            steps {
                sh 'curl -f http://host.docker.internal:5000'
            }
        }
    }

    post {
        always {
            sh 'docker stop app-test || true'
            sh 'docker rm app-test || true'
        }
    }
}
