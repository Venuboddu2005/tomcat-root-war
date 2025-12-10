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
                    mvn jetty:run &
                """
            }
        }

    
    }
}
