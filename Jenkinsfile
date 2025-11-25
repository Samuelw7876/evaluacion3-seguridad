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

        stage('Security Scan - Bandit') {
            steps {
                sh '''
                    echo "Ejecutando Bandit..."
                    echo "Workspace real: $WORKSPACE"
                    echo "Contenido del workspace:"
                    ls -R $WORKSPACE

                    docker run --rm \
                        -v $WORKSPACE/app:/scan \
                        python:3.11-slim sh -c "
                            pip install --quiet bandit && \
                            bandit -r /scan
                        " || true
                '''
            }
        }

        stage('Security Scan - pip-audit') {
            steps {
                sh '''
                    echo "Ejecutando pip-audit dentro de contenedor..."
                    docker run --rm \
                        -v $(pwd)/app:/scan \
                        python:3.11-slim sh -c "
                            pip install --quiet pip-audit && \
                            pip-audit -r /scan/requirements.txt || true
                        "
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Docker host: ${DOCKER_HOST}"
                sh 'docker version'
                sh 'docker build -t evaluacion3-app ./app'
            }
        }

        stage('Security Scan - Trivy') {
            steps {
                sh '''
                    echo "Ejecutando escaneo de seguridad con Trivy..."
                    docker run --rm \
                        -v /var/run/docker.sock:/var/run/docker.sock \
                        aquasec/trivy image evaluacion3-app || true
                '''
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
