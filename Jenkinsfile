pipeline {
    agent any

    options {
        skipStagesAfterUnstable()
    }

    tools {
        maven '3.9.11'
    }

    environment {
        // Nexus repo info
        NEXUS_URL = "http://localhost:8081/repository/maven-releases/"
        NEXUS_CREDENTIALS = "nexus-creds"
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

        stage('Upload to Nexus') {
            steps {
                dir('complete') {
                    withCredentials([usernamePassword(credentialsId: "${NEXUS_CREDENTIALS}",
                                                     usernameVariable: 'NEXUS_USER',
                                                     passwordVariable: 'NEXUS_PASS')]) {

                        sh """
                        mvn deploy:deploy-file \
                            -Durl=${NEXUS_URL} \
                            -DrepositoryId=nexus \
                            -Dfile=target/demo-0.0.1-SNAPSHOT.jar \
                            -DgroupId=com.example \
                            -DartifactId=demo \
                            -Dversion=0.0.1-SNAPSHOT \
                            -Dpackaging=jar \
                            -DgeneratePom=true \
                            -Dusername=$NEXUS_USER \
                            -Dpassword=$NEXUS_PASS
                        """
                    }
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
