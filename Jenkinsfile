pipeline {
    agent any

    environment {
        // Define environment variables
        SONARQUBE_SERVER = 'Sonar'    // Replace with your SonarQube server name in Jenkins
        DOCKER_REGISTRY = 'yuvikedari' // Replace with your Docker registry
        IMAGE_NAME = 'java-app'                 // Replace with your image name
        IMAGE_TAG = 'latest'                    // Or dynamically use GIT_COMMIT for unique tagging
    }

    stages {
        stage('Code Fetch') {
            steps {
                echo 'Fetching code from SCM...'
                checkout scm
            }
        }

        stage('Compile') {
            steps {
                echo 'Compiling the code...'
                sh 'mvn compile'
            }
        }

        stage('Testing') {
            steps {
                echo 'Running tests...'
                sh 'mvn test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo 'Performing SonarQube analysis...'
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Build Artifact') {
            steps {
                echo 'Building the artifact...'
                sh 'mvn package'
            }
            post {
                success {
                    archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh """
                    docker build -t ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} .
                """
            }
        }

        stage('Scan Docker Image') {
            steps {
                echo 'Scanning Docker image...'
                // Example: Using Trivy for image scanning
                sh """
                    trivy image ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                echo 'Pushing Docker image to registry...'
                withDockerRegistry([credentialsId: 'docker-credentials', url: "https://${DOCKER_REGISTRY}"]) {
                    sh """
                        docker push ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Update Manifest File') {
            steps {
                echo 'Updating manifest file...'
                // Replace this block with the actual logic to update the manifest file
                sh """
                    echo "image: ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}" > updated-manifest.yaml
                    # Commit and push the updated manifest to another repo
                    git config user.name "jenkins"
                    git config user.email "jenkins@yourdomain.com"
                    git add updated-manifest.yaml
                    git commit -m "Updated manifest with new image: ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                    git push origin main
                """
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Please check the logs for errors.'
        }
    }
}
