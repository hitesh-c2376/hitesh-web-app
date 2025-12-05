pipeline {
    agent any

    environment {
        IMAGE_NAME = "simple_webapp"
        IMAGE_TAG  = "latest"
        DOCKER_IMAGE = "simple_webapp:latest"
        TRIVY_REPORT_JSON = "trivy-report.json"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/hitesh-c2376/hitesh-web-app.git'
            }
        }

        stage('Environment Variables') {
            steps {
                sh '''
                echo "Docker Image: ${IMAGE_NAME}:${IMAGE_TAG}"
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                echo "🔨 Building Docker Image..."
                docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Trivy Scan') {
            steps {
                sh '''
                echo "🔍 Running Trivy Vulnerability Scan..."
                trivy image ${DOCKER_IMAGE} || echo "Trivy scan completed with possible findings"
                '''
            }
        }

        stage('Trivy JSON Report') {
            steps {
                sh '''
                echo "📄 Generating Trivy JSON Report..."
                trivy image --format json ${DOCKER_IMAGE} --output ${TRIVY_REPORT_JSON}
                '''
            }
            post {
                success {
                    archiveArtifacts artifacts: "${TRIVY_REPORT_JSON}", fingerprint: true
                }
            }
        }

        stage('Trivy HTML Report') {
            steps {
                sh '''
                echo "🌐 Generating Trivy HTML Report..."

                # Download template if missing
                if [ ! -f html.tpl ]; then
                    curl -L -o html.tpl https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl
                fi

                # Generate HTML
                trivy image --format template --template "html.tpl" ${DOCKER_IMAGE} --output trivy-report.html
                '''
            }
            post {
                success {
                    archiveArtifacts artifacts: 'trivy-report.html', fingerprint: true
                }
            }
        }

        stage('Finish') {
            steps {
                echo "🎉 Pipeline completed successfully!"
                echo "🟢 Docker Build Complete"
                echo "🟢 Trivy Scan Complete"
                echo "🟢 Reports Archived"
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished (post block).'
        }
    }
}
