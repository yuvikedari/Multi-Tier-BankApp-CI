pipeline {
    agent any
        docker {
          image 'abhishekf5/maven-abhishek-docker-agent:v1'
          args '--user root -v /var/run/docker.sock:/var/run/docker.sock' // mount Docker socket to access the host's Docker daemon
        }
      }

    stages {
        stage('Code Fetch') {
            steps {
                sh 'echo passed'
                git branch: 'main', url: 'https://github.com/yuvikedari/Multi-Tier-BankApp-CI.git'
            }
        }

        stage('Build and Test') {
            steps {
                sh 'ls -ltr'
                // build the project and create a JAR file
                sh 'cd Multi-Tier-BankApp-CI && mvn clean package'
            }
        }

        stage('Testing') {
            steps {
                echo 'Running tests...'
                sh 'mvn test'
            }
        }

        stage('Static Code Analysis') {
          environment {
            SONAR_URL = "http://3.90.51.121:9000/"
          }
          steps {
            withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
              sh 'cd Multi-Tier-BankApp-CI && mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
            }
          }
        }

         stage('Build and Push Docker Image') {
          environment {
            DOCKER_IMAGE = "yuvikedari/my-bank-app:${BUILD_NUMBER}"
            // DOCKERFILE_LOCATION = "java-maven-sonar-argocd-helm-k8s/spring-boot-app/Dockerfile"
            REGISTRY_CREDENTIALS = credentials('docker-cred')
          }
          steps {
            script {
                sh 'cd Multi-Tier-BankApp-CI && docker build -t ${DOCKER_IMAGE} .'
                def dockerImage = docker.image("${DOCKER_IMAGE}")
                docker.withRegistry('https://index.docker.io/v1/', "docker-cred") {
                dockerImage.push()
                }
            }
          }
        }

        
        stage('Update Deployment File') {
        environment {
            GIT_REPO_NAME = "Multi-Tier-BankApp-CD"
            GIT_USER_NAME = "yuvikedari"
        }
        steps {
            withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                sh '''
                    git config user.email "yuvikedari@gmail.com"
                    git config user.name "Yuvi Kedari"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" /deployment.yml
                    git add Multi-Tier-BankApp-CD/bankapp/bankapp-ds.yml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                '''
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
