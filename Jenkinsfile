pipeline {
    agent any

    tools {
        jdk "OracleJDK8"
        maven "MAVEN3"
    }

    environment {
        scannerHome = tool 'sonar4'
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "172.31.82.67:8081"
        NEXUS_REPOSITORY = "vprofile-repo"
        NEXUS_CREDENTIAL_ID = "Nexus-token"
        NEXUS_PASS = credentials('nexuspass')
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
                nexusUrl: NEXUS_URL,
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

        stage('Ansible Deploy to staging') {
            steps {
                ansiblePlaybook([
                    playbook: 'ansible/site.yml',
                    inventory: 'ansible/stage-inventory',
                    installation: 'ansible',
                    credentialsId: 'app01',
                    colorized: true,
                    disableHostKeyChecking: true,
                    extraVars : [
                        USER: 'admin',
                        PASS: NEXUS_PASS,
                        nexusip: '34.238.114.178',
                        reponame: 'vprofile-repo',
                        groupid: 'QA',
                        time: "${env.BUILD_TIMESTAMP}",
                        build: "${env.BUILD_ID}",
                        artifactId: 'vprofile-app',
                        vprofile_version: "vprofile-app-${env.BUILD_ID}-${env.BUILD_TIMESTAMP}.war"
                    ]
                ])
            }
        }
    }
}