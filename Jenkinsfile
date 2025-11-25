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
                sh '''
                    echo " Levantando la app para ZAP..."
                    docker run -d --name zap-app -p 5000:5000 evaluacion3-app
                    sleep 8

                    echo " Ejecutando OWASP ZAP..."

                    mkdir -p ${WORKSPACE}/zap_reports

                    docker run --rm \
                        --network host \
                        -v "${WORKSPACE}/zap_reports:/zap/wrk" \
                        ghcr.io/zaproxy/zaproxy:stable \
                        zap-baseline.py -t http://localhost:5000 -r zap_report.html || true

                    echo " Reporte ZAP generado en:"
                    echo "${WORKSPACE}/zap_reports/zap_report.html"

                    docker stop zap-app || true
                    docker rm zap-app || true
                '''
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
                        -v "${WORKSPACE}/app:/workspace" \
                        python:3.11-slim \
                        sh -c "pip install --quiet pip-audit && pip-audit -r /workspace/requirements.txt || true"

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
