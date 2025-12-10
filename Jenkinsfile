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
                        -Dspring.docker.compose.skip.in-tests=true \
                        -Dtest=*Tests,!*IntegrationTests,*Test
                """
            }
        }
    }
}
