pipeline {
    agent {
        docker { image 'eclipse-temurin:17' }
    }
    stages {
        stage('Commit') {
            steps {
                echo "Commit stage"
                sh "./mvnw -B package -Dbuild.number=${BUILD_NUMBER}"
            }
        }    
        stage('Acceptance') {
            steps {
                echo "Acceptance stage"
                sh "./mvnw -B integration-test -Dbuild.number=${BUILD_NUMBER}"
            }
        }
    }
}