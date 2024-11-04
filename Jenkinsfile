pipeline {
    agent any
    parameters {
        choice(name: 'DEPLOY_ENV', choices: ['blue', 'green'], description: 'Choose which environment to deploy: Blue or Green')
        choice(name: 'DOCKER_TAG', choices: ['blue', 'green'], description: 'Choose the Docker image tag for the deployment')
    }
    environment {
        IMAGE_NAME = "911167902479.dkr.ecr.us-east-1.amazonaws.com/bankapp"
        TAG = "${params.DOCKER_TAG}"
        SCANNER_HOME = tool 'sonar-scanner'
        JAVA_HOME = tool name: 'java-17'
        MAVEN_HOME = tool 'maven3'
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'new-qa', credentialsId: 'git-cred', url: 'https://github.com/Jaanu2429/Blue-Green-Deployment.git'
            }
        }
        stage('Maven Build') {
            steps {
                script {
                    sh "mvn package -DskipTests"
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=nodejsmysql -Dsonar.projectName=nodejsmysql -Dsonar.java.binaries=target/classes"
                }
            }
        }
        stage('Login to Amazon ECR') {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'aws-access-key', variable: 'AWS_ACCESS_KEY_ID'),
                        string(credentialsId: 'aws-secret-key', variable: 'AWS_SECRET_ACCESS_KEY')
                    ]) {
                        sh '''
                            aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                            aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                            aws configure set default.region us-east-1
                            aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 911167902479.dkr.ecr.us-east-1.amazonaws.com/bankapp
                        '''
                    }
                }
            }
        }
        stage('Docker build') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_NAME}:${TAG} ."
                }
            }
        }
        stage('Docker Push Image') {
            steps {
                script {
                    sh "docker push ${IMAGE_NAME}:${TAG}"
                }
            }
        }
    }
}
