pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('sonar-token')
        DOCKER_IMAGE = "my-python-app:latest"
        VENV = ".venv"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/hitesh-c2376/my-python-app.git'
            }
        }

        stage('Setup Python Environment') {
            steps {
                sh '''
                python3 -m venv ${VENV}
                . ${VENV}/bin/activate
                pip install --upgrade pip
                pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                . ${VENV}/bin/activate
                pytest tests/ --disable-warnings --maxfail=1
                '''
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                    . ${VENV}/bin/activate
                    sonar-scanner \
                      -Dsonar.projectKey=my-python-app \
                      -Dsonar.sources=. \
                      -Dsonar.host.url=$SONAR_HOST_URL \
                      -Dsonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('Trivy Scan') {
            steps {
                sh '''
                trivy image --format json --output trivy-report.json ${DOCKER_IMAGE}
                trivy image --format table ${DOCKER_IMAGE}
                '''
                archiveArtifacts artifacts: 'trivy-report.json', fingerprint: true
            }
        }

        stage('Docker Scout Scan') {
            steps {
                sh '''
                docker scout quickview ${DOCKER_IMAGE} || echo "Scout warnings"
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                '''
            }
        }
    }

    post {
        always {
            echo "Pipeline completed."
        }
    }
}
