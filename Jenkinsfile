pipeline {
    agent any
    parameters {
        choice(name: 'DEPLOY_ENV', choices: ['blue', 'green'], description: 'Choose which environment to deploy: Blue or Green')
        choice(name: 'DOCKER_TAG', choices: ['blue', 'green'], description: 'Choose the Docker image tag for the deployment')
    }
    environment {
        IMAGE_NAME = "463470944171.dkr.ecr.us-east-1.amazonaws.com/ecrrepo1"
        TAG = "${params.DOCKER_TAG}"
        JAVA_HOME = tool name: 'java-17'
        MAVEN_HOME = tool 'maven3'
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'dev', credentialsId: 'git-cred', url: 'https://github.com/Jaanu2429/Blue-Green-Deployment.git'
            }
        }
        stage('Maven Build') {
            steps {
                script {
                    sh "mvn package -DskipTests"
                }
            }
        }
        
        stage('Login to Amazon ECR') {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'aws_access_key', variable: 'AWS_ACCESS_KEY_ID'),
                        string(credentialsId: 'aws_secret_key', variable: 'AWS_SECRET_ACCESS_KEY')
                    ]) {
                        sh '''
                            export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                            export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                            export AWS_DEFAULT_REGION=us-east-1
                            
                            aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 463470944171.dkr.ecr.us-east-1.amazonaws.com/ecrrepo1
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
     
