pipeline {
    agent any

    environment {
        BUILD_VERSION = "1.0.${env.BUILD_NUMBER}-SNAPSHOT"
    }

    stages {

        stage("Build code") {
            steps {
                sh """
                    mvn clean package
                """
            }
        }

        stage("Run Jetty") {
            steps {
                sh """
                    nohup mvn jetty:run > jetty.log 2>&1 &
                    sleep 5
                """
            }
        }

        stage("Run unit tests only") {
            steps {
                sh """
                    mvn test \
                        -Dspring.docker.compose.skip-in-tests=true \
                        -Dtest=*Tests,!*IntegrationTests,*Test \
                        -DfailIfNoTests=false
                """
            }
        }

        stage("Sonar Scan") {
            steps {
                script {
                    withSonarQubeEnv('sonar-server-01') {
                        sh """
                            sonar-scanner \
                                -Dsonar.projectKey=tomcat-root-war \
                                -Dsonar.projectName=tomcat-root-war \
                                -Dsonar.sources=src/main/java \
                                -Dsonar.java.binaries=target/classes \
                                -Dsonar.sourceEncoding=UTF-8 \
                                -Dsonar.host.url=$SONAR_HOST_URL
                        """
                    }
                }
            }
        }

        stage("Sonar Quality Gate") {
            steps {
                script {
                    timeout(time: 2, unit: 'MINUTES') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "❌ Quality Gate Failed: ${qg.status}"
                        }
                    }
                }
            }
        }

        stage("Upload-Artifacts-Nexus") {
            steps {
                script {
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: 'nexus:8081',

                        repository: 'maven-snapshots',  // ✅ SNAPSHOT repo allows redeploy
                        credentialsId: 'nexus-creds',

                        groupId: 'com.akalashnykov',
                        version: BUILD_VERSION,          // ✅ Auto version: 1.0.<build>-SNAPSHOT

                        artifacts: [
                            [
                                artifactId: 'tomcat-root-war',
                                classifier: '',
                                file: 'target/ROOT.war',
                                type: 'war'
                            ]
                        ]
                    )
                }
            }
        }

        stage("Deploy-Dev") {
            steps {
                echo "Deploying to DEV servers..."
            }
        }

        stage("Deploy-UAT") {
            steps {
                echo "Deploying to UAT servers..."
            }
        }

        stage("Deploy-PROD") {
            steps {
                echo "Deploying to PROD servers..."
            }
        }
    }
}
