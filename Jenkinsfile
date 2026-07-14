pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    environment {
        APP_NAME = 'spring-petclinic'
        APP_DIR = 'application/spring-petclinic'
        SONARQUBE_ENV = 'SonarQube'
    }

    options {
        timestamps()
        ansiColor('xterm')
        disableConcurrentBuilds()
        buildDiscarder(logRotator(
            numToKeepStr: '20',
            artifactNumToKeepStr: '10'
        ))
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Project') {
            steps {
                sh '''
                    pwd
                    ls -la
                    ls -la ${APP_DIR}
                '''
            }
        }

        stage('Build') {
            steps {
                dir("${APP_DIR}") {
                    sh 'mvn clean compile'
                }
            }
        }

        stage('Unit Tests') {
            steps {
                dir("${APP_DIR}") {
                    sh 'mvn test'
                }
            }
        }

        stage('Package') {
            steps {
                dir("${APP_DIR}") {
                    sh 'mvn package -DskipTests'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                dir("${APP_DIR}") {
                    withSonarQubeEnv("${SONARQUBE_ENV}") {
                        sh '''
                        mvn sonar:sonar \
                        -Dsonar.projectKey=spring-petclinic \
                        -Dsonar.projectName=Spring-PetClinic
                        '''
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

    }

    post {

        always {
            junit '**/target/surefire-reports/*.xml'
            archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
        }

        success {
            echo 'Pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed.'
        }
    }
}
