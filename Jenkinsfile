pipeline {
    agent any
    tools {
        jdk 'jdk21'
        maven 'maven'
    }
    triggers {
        pollSCM("H/2 * * * *")
    }
    environment {
        APP_NAME = "spring-petclinic"
        AWS_REGION = "us-east-1"
        AWS_ACCOUNT_ID = "016338413637"
        ECR_REPOSITORY = "spring-petclinic"
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        SONARQUBE_SERVER = "SonarQube"
        IMAGE_TAG = "${BUILD_NUMBER}"
        EC2_HOST = "44.211.119.130"
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
                    -t ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG} \
                    -t ${ECR_REGISTRY}/${ECR_REPOSITORY}:latest \
                    .
                    '''
                }
            }
        }
        stage('Push to ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-prod']]) {
                    sh '''
                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}
                    docker push ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}
                    docker push ${ECR_REGISTRY}/${ECR_REPOSITORY}:latest
                    '''
                }
            }
        }
        stage('Deploy to EC2') {
            steps {
                sshagent(credentials: ['ec2-prod-ssh']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ubuntu@${EC2_HOST} "
                        aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY} &&
                        docker pull ${ECR_REGISTRY}/${ECR_REPOSITORY}:latest &&
                        docker stop spring-petclinic || true &&
                        docker rm spring-petclinic || true &&
                        docker run -d --name spring-petclinic --network petclinic-network --restart unless-stopped ${ECR_REGISTRY}/${ECR_REPOSITORY}:latest
                    "
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
