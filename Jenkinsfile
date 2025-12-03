pipeline {
    agent any

    options {
        skipStagesAfterUnstable()
    }

    tools {
        maven '3.9.11'
    }

    environment {
        // Use the Nexus container hostname, not localhost
        NEXUS_URL = 'http://nexus:8081/repository/maven-releases/'
        NEXUS_CREDENTIALS = 'nexus-creds'
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

        stage('Upload to Nexus') {
            steps {
                dir('complete') {
                    withCredentials([usernamePassword(
                        credentialsId: "${NEXUS_CREDENTIALS}",
                        usernameVariable: 'NEXUS_USER',
                        passwordVariable: 'NEXUS_PASS'
                    )]) {
                        script {
                            def jarFile = sh(
                                script: "ls target/*.jar | grep -v original | head -n 1",
                                returnStdout: true
                            ).trim()
                            echo "Deploying JAR: ${jarFile}"

                            sh """
                            mvn deploy:deploy-file \
                                -Durl=$NEXUS_URL \
                                -DrepositoryId=nexus \
                                -Dfile=${jarFile} \
                                -DgroupId=com.example \
                                -DartifactId=spring-boot-complete \
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
        }

    }

    post {
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo 'Build and deployment successful!'
        }
        failure {
            echo 'Build or deployment failed!'
        }
    }
}
