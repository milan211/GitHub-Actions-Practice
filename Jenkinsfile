// This Jenkins file is for Eureka Deployment

pipeline {
    agent {
        label 'k8s-slave'
    }
    environment {
        APPLICATION_NAME = "eureka"
    }

    stages {
        stage ('Build'){
            // This is Where Build for Eureka application happens
            steps {
                echo "Building ${env.APPLICATION_NAME} Application"
                sh 'mvn clean package'
            }
        }
    }
}