pipeline {

    environment {
            DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
            VERSION_NUMBER = sh (
                        script: './mvnw help:evaluate -Dexpression=project.version -Dbuild.number=${BUILD_NUMBER} -q -DforceStdout',
                        returnStdout: true).trim()                
            IMAGE_NAME = "nemesbalint/employees-github:${VERSION_NUMBER}"
            SONAR_CREDENTIALS = credentials('sonar-credentials')
    }

    agent {
        dockerfile {
            filename 'Dockerfile.build'
            args '-e DOCKER_CONFIG=./docker'
        }
    }

    stages {
        stage('Commit') {
            steps {
                echo "Commit stage"
                sh "./mvnw -B clean package -Dbuild.number=${BUILD_NUMBER}"
            }
        }    
        stage('Acceptance') {
            steps {
                echo "Acceptance stage"
                sh "./mvnw -B integration-test -Dbuild.number=${BUILD_NUMBER}"
            }
        }
        stage('Docker') {
            steps {
                echo "Docker"
                sh "docker build -f Dockerfile.layered -t ${IMAGE_NAME} ."
                sh "echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u=${DOCKERHUB_CREDENTIALS_USR} --password-stdin"
                sh "docker push ${IMAGE_NAME}"
                sh "docker tag ${IMAGE_NAME} nemesbalint/employees-github:latest"
                sh "docker push nemesbalint/employees-github:latest"                
            }
        }
        stage('E2E API') {            
            steps {
                dir('employees-postman') {
                    sh 'rm -rf reports'
                    sh 'mkdir reports'
                    sh 'docker compose -f docker-compose.yaml -f docker-compose.jenkins.yaml up --abort-on-container-exit'   
                    archiveArtifacts artifacts: 'reports/*.html', fingerprint: true                 
                }
            }
        }
        stage('Code quality') {
            steps {
                sh "./mvnw sonar:sonar -Dsonar.host.url=http://host.docker.internal:9000 -Dsonar.login=${SONAR_CREDENTIALS_PSW}"
            }
        }
    }
}