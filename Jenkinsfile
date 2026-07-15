pipeline {
    agent any
    tools {
        jdk 'jdk21'
        maven 'maven'
    }
    environment {
        APP_NAME = "spring-petclinic"
        AWS_REGION = "eu-west-1"
        ECR_REPOSITORY = "spring-petclinic"
        SONARQUBE_SERVER = "SonarQube"
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKER_IMAGE = "stephenesu/${APP_NAME}"
    }
    options {
        timestamps()
        ansiColor('xterm')
        buildDiscarder(logRotator(numToKeepStr: '20'))
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                dir('application/spring-petclinic') {
                    sh 'mvn clean compile'
                }
            }
        }
        stage('Unit Test') {
            steps {
                dir('application/spring-petclinic') {
                    sh 'mvn test'
                }
            }
        }
        stage('SonarQube Analysis') {
            environment {
                scannerHome = tool 'SonarQube'
            }
            steps {
                dir('application/spring-petclinic') {
                    withSonarQubeEnv('SonarQube') {
                        withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN')]) {
                            sh '''
                            ${scannerHome}/bin/sonar-scanner \
                              -Dsonar.projectKey=spring-petclinic \
                              -Dsonar.projectName=Spring-Petclinic \
                              -Dsonar.sources=src \
                              -Dsonar.java.binaries=target/classes \
                              -Dsonar.host.url=$SONAR_HOST_URL \
                              -Dsonar.login=$SONAR_TOKEN
                            '''
                        }
                    }
                }
            }
        }
        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Package') {
            steps {
                dir('application/spring-petclinic') {
                    sh 'mvn package -DskipTests'
                }
            }
        }
        stage('Docker Build') {
            steps {
                dir('application/spring-petclinic') {
                    sh '''
                    docker build \
                    -t ${DOCKER_IMAGE}:${IMAGE_TAG} \
                    -t ${DOCKER_IMAGE}:latest \
                    .
                    '''
                }
            }
        }
        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVar: 'DOCKER_USER', passwordVar: 'DOCKER_PASS')]) {
                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    docker push ${DOCKER_IMAGE}:${IMAGE_TAG}
                    docker push ${DOCKER_IMAGE}:latest
                    docker logout
                    '''
                }
            }
        }
    }
    post {
        success {
            echo "Pipeline completed successfully."
        }
        failure {
            echo "Pipeline failed."
        }
        always {
            cleanWs()
        }
    }
}
