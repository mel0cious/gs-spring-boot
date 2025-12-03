pipeline {
    agent any

    options {
        skipStagesAfterUnstable()
    }

    tools {
        maven '3.9.11'
    }

    environment {
        NEXUS_URL = 'http://nexus:8081/repository/maven-releases/' // Nexus URL from Jenkins container
        NEXUS_CREDENTIALS = 'nexus-creds' // Jenkins credentials ID for Nexus
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
                    sh 'mvn clean test -e -X' // Full error and debug output
                }
            }
        }

        stage('Build and Package') {
            steps {
                dir('complete') {
                    sh 'mvn clean package -DskipTests -e -X'
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
                            // Find the built JAR dynamically
                            def jarFile = sh(script: "ls target/*.jar | grep -v original | head -n 1", returnStdout: true).trim()
                            echo "Deploying JAR: ${jarFile}"

                            // Generate temporary settings.xml with credentials
                            writeFile file: 'temp-settings.xml', text: """
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                              https://maven.apache.org/xsd/settings-1.0.0.xsd">
  <servers>
    <server>
      <id>nexus</id>
      <username>${NEXUS_USER}</username>
      <password>${NEXUS_PASS}</password>
    </server>
  </servers>
</settings>
"""

                            // Deploy using Maven with temp settings.xml
                            sh """
                            mvn deploy:deploy-file -e -X \
                                -Durl=${NEXUS_URL} \
                                -DrepositoryId=nexus \
                                -Dfile=${jarFile} \
                                -DgroupId=com.example \
                                -DartifactId=spring-boot-complete \
                                -Dversion=0.0.1-SNAPSHOT \
                                -Dpackaging=jar \
                                -DgeneratePom=true \
                                -s temp-settings.xml
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
