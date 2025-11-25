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

        stage('Debug Volume') {
            steps {
                sh """
                    echo 'Contenido real del workspace desde Jenkins:'
                    ls -R ${WORKSPACE}/app

                    echo '------------ Ejecutando prueba de montaje ------------'

                    docker run --rm \
                        -v ${WORKSPACE}/app:/scan \
                        python:3.11-slim sh -c "echo 'Contenido dentro del contenedor:' && ls -R /scan || true"
                """
            }
        }

        stage('Security Scan - OWASP ZAP') {
            steps {
                echo "Iniciando escaneo DAST con OWASP ZAP"

                sh """
                    docker pull ghcr.io/zaproxy/zaproxy:stable || true

                    docker run --rm \
                        -v \$(pwd)/zap_reports:/zap/reports \
                        ghcr.io/zaproxy/zaproxy:stable \
                        zap-baseline.py \
                        -t http://localhost:5000 \
                        -r zap_report.html || true
                """

                echo "Informe generado en zap_reports/zap_report.html"
            }
        }



        stage('Security Scan - Bandit') {
            steps {
                script {
                    echo "Workspace real: ${env.WORKSPACE}"

                    sh """
                        docker run --rm \
                            -v ${env.WORKSPACE}/app:/scan \
                            python:3.11-slim sh -c '
                                echo "Archivos dentro del contenedor:";
                                ls -R /scan;
                                pip install --quiet bandit &&
                                bandit -r /scan -lll
                            '
                    """
                }
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
