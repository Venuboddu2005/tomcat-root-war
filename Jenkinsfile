pipeline {
    agent any

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

                      stage("Upload-Artifacts-Nexus") {
                          steps {
                                script {
                                     nexusArtifactUploader(
                                     nexusVersion: 'nexus3',
                                      protocol: 'http',
                                      nexusUrl: 'nexus:8081',   // FIXED
                                       repository: 'maven-snapshots',
                                       credentialsId: 'nexus-creds',

                                       groupId: 'org.springframework.samples',
                                       version: '4.0.0-SNAPSHOT',

                                       artifacts: [
                                                [
                                        artifactId: 'tomcat-root-war',
                                        classifier: '',
                                        file: 'target/tomcat-root-war-4.0.0-SNAPSHOT.jar',
                                         type: 'jar'
                    ]
                ]
            )
        }
    }
}

                }
            }
        }
    }
}
