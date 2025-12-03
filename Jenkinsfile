pipeline {
    agent any

    options {
        skipStagesAfterUnstable()
    }

    tools {
        maven '3.9.11'
    }

    stages {

        stage('Checkout Source Code') {
            steps {
                git branch: 'main', url: 'https://github.com/mel0cious/gs-spring-boot.git'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn --version'
                dir('complete') {
                    sh 'mvn clean test'
                }
            }
        }

        stage('Build and Package') {
            steps {
                dir('complete') {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'complete/target/*.jar', fingerprint: true
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo 'Build successful!'
        }
        failure {
            echo 'Build failed!'
        }
    }
}
