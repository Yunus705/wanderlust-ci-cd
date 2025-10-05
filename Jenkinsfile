pipeline{
    agent any
    environment{
        SONAR_HOME= tool "Sonar"
        DOCKER_HUB_USER = 'yunus05'
        DOCKER_HUB_PASS = credentials('dockerhub-cred-id')
    }
    stages{
        stage("Clone Code from GitHub"){
            steps{
                git url: "https://github.com/Yunus705/wanderlust-ci-cd.git", branch: "main"
            }
        }
        stage("SonarQube Quality Analysis"){
            steps{
                withSonarQubeEnv("Sonar"){
                    sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=wanderlust -Dsonar.projectKey=wanderlust"
                }
            }
        }
        stage("OWASP Dependency Check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'dc'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Build Docker Images') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_HUB_USER}/frontend:latest ./frontend"
                    sh "docker build -t ${DOCKER_HUB_USER}/backend:latest ./backend"
                }
            }
        }
        stage("Trivy Image Scan"){
            steps{
                sh "trivy image ${DOCKER_HUB_USER}/frontend:latest || true"
                sh "trivy image ${DOCKER_HUB_USER}/backend:latest || true"
            }
        }
        stage('Push to Docker Hub') {
            steps {
                script {
                    sh "echo $DOCKER_HUB_PASS | docker login -u $DOCKER_HUB_USER --password-stdin"
                    sh "docker push ${DOCKER_HUB_USER}/frontend:latest"
                    sh "docker push ${DOCKER_HUB_USER}/backend:latest"
                }
            }
        }
        stage("Deploy using Docker compose"){
            steps{
                sh 'docker-compose down || true'
                sh "docker-compose up -d"
            }
        }
    }
}
