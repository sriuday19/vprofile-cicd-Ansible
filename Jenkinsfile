pipeline {
    agent any

    tools {
        jdk "OracleJDK17"
        maven "MAVEN3"
    }

    environment {
        scannerHome = tool 'sonar5'
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "http://34.238.114.178:8081"
        NEXUS_REPOSITORY = "vprofile-app"
        NEXUS_CREDENTIAL_ID = "nexus-token"
    }

    stages {

        stage('Build code') {

            steps {
                sh 'mvn clean install'
            } 

            post {
                success {
                    echo 'Archieving the artifact'
                    archiveArtifacts artifacts: "**/target/*.war"
                }
            }
        }

        stage('Test') {

            steps {
                sh 'mvn test'
            }
        }

        stage('Checkstyle Analysis') {

            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }

        stage('Code Analysis withe Sonqrqube') {

            steps {

                withSonarQubeEnv("sonar") {
                    sh "${scannerHome}/bin/sonar-scanner \
                    -Dsonar.projectKey=vprofile \
                    -Dsonar.projectName=vprofile-app \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.sources=src/ \
                    -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                    -Dsonar.junit.reportsPath=target/surefire-reports/ \
                    -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                    -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml "
                    
                }
            }
        }

        stage('Quality-gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Uploading the artifasct to nexus') {
            steps {
                nexusArtifactUploader(
                nexusVersion: NEXUS_VERSION,
                protocol: NEXUS_PROTOCOL,
                nexusUrl: NEXUS_PROTOCOL,
                groupId: 'QA',
                version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                repository: NEXUS_REPOSITORY,
                credentialsId: NEXUS_CREDENTIAL_ID,
                artifacts: [
                [artifactId: 'vprofile-app',
                 classifier: '',
                 file: 'target/vprofile-v2.war',
                 type: 'war']
                ]
            )
            }
        }
    }
}