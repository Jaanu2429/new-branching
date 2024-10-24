pipeline {

    agent any

    parameters {

        choice(name: 'DEPLOY_ENV', choices: ['blue', 'green'], description: 'Choose which environment to deploy: Blue or Green')

        choice(name: 'DOCKER_TAG', choices: ['blue', 'green'], description: 'Choose the Docker image tag for the deployment')

    }

    environment {

        IMAGE_NAME = "anusha2429/bankapp"

        TAG = "${params.DOCKER_TAG}"  // The image tag now comes from the parameter

        SCANNER_HOME = tool 'sonar-scanner'

        JAVA_HOME = tool name: 'java-17' 

        MAVEN_HOME = tool 'maven3'  // Specify the Maven tool

    }

    stages {

        stage('Git Checkout') {

            steps {

                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/Jaanu2429/Blue-Green-Deployment.git'

            }

        }

        // Maven Build Stage

        stage('Maven Build') {

            steps {

                script {

                    // Skipping tests with -DskipTests

                    sh "mvn package -DskipTests"

                }

            }

        }

        // SonarQube Analysis Stage

        stage('SonarQube Analysis') {

            steps {

                withSonarQubeEnv('sonar') {

                    sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=nodejsmysql -Dsonar.projectName=nodejsmysql -Dsonar.java.binaries=target/classes"

                }

            }

        }

        // Docker build stage

        stage('Docker build') {

            steps {

                script {

                    withDockerRegistry(credentialsId: 'docker-cred') {

                        sh "docker build -t ${IMAGE_NAME}:${TAG} ."

                    }

                }

            }

        }

        // Docker Push Image

        stage('Docker Push Image') {

            steps {

                script {

                    withDockerRegistry(credentialsId: 'docker-cred') {

                        sh "docker push ${IMAGE_NAME}:${TAG}"

                    }

                }

            }

        }

    }

}

 
