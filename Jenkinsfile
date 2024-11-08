pipeline {
    agent any

    environment {
        GITHUB_REPO = 'https://github.com/your-repo/maven-project.git'
        GIT_BRANCH = 'main'
        SONARQUBE_SERVER = 'SonarQube'  // Configure in Jenkins -> Manage Jenkins -> Configure System -> SonarQube servers
        NEXUS_URL = 'https://your-nexus-repo-url'
        NEXUS_CREDENTIALS_ID = 'nexus-credentials-id'
        DOCKER_HUB_REPO = 'your-dockerhub-username/your-image-name'
        DOCKER_HUB_CREDENTIALS_ID = 'dockerhub-credentials-id'
        EKS_CLUSTER_NAME = 'your-eks-cluster-name'
        EKS_REGION = 'us-east-1'
    }

    stages {
        stage('Clone Repo') {
            steps {
                echo 'Cloning GitHub repository...'
                git branch: "${GIT_BRANCH}", url: "${GITHUB_REPO}"
            }
        }

        stage('Maven Build') {
            steps {
                echo 'Building the Maven project...'
                sh 'mvn clean package'
            }
        }

        stage('Code Review') {
            steps {
                echo 'Running SonarQube analysis...'
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Nexus Upload') {
            steps {
                echo 'Uploading WAR file to Nexus...'
                script {
                    def warFile = sh(script: 'ls target/*.war', returnStdout: true).trim()
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: "${NEXUS_URL}",
                        groupId: 'com.example',
                        version: '1.0.0',
                        repository: 'maven-releases',
                        credentialsId: "${NEXUS_CREDENTIALS_ID}",
                        artifacts: [
                            [artifactId: 'maven-app', classifier: '', file: warFile, type: 'war']
                        ]
                    )
                }
            }
        }

        stage('Docker Image Build') {
            steps {
                echo 'Building Docker image...'
                sh 'docker build -t ${DOCKER_HUB_REPO}:latest .'
            }
        }

        stage('Docker Image Push') {
            steps {
                echo 'Pushing Docker image to DockerHub...'
                withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CREDENTIALS_ID}", passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                    sh 'docker push ${DOCKER_HUB_REPO}:latest'
                }
            }
        }

        stage('Configure kubectl for EKS') {
            steps {
                echo 'Configuring kubectl for EKS...'
                withAWS(region: "${EKS_REGION}") {
                    sh """
                    aws eks update-kubeconfig --region ${EKS_REGION} --name ${EKS_CLUSTER_NAME}
                    kubectl version --short
                    """
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                echo 'Deploying to EKS...'
                sh 'kubectl apply -f k8s/deployment.yaml'
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Please check the logs.'
        }
    }
}

