ppipeline {
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
                    mvn jetty:run &
                """
            }
        }

        stage("Run unit tests only") {
            steps {
                sh """
                    mvn test \
                        -Dspring.docker.compose.skip.in-tests=true \
                        -Dtest='*Tests,!*IntegrationTests,*Test'
                """
            }
        }
    }
}
