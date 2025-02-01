// This Jenkins file is for Eureka Deployment

pipeline {
    agent {
        label 'k8s-slave'
    }

    // tools configured in jenkins-master
    tools {
        maven 'Maven-3.8.8'
        jdk 'JDK-17'
    }
    environment {
        APPLICATION_NAME = "eureka"
    }

    stages {
        stage ('Build'){
            // This is Where Build for Eureka application happens
            steps {
                echo "Building ${env.APPLICATION_NAME} Application"
                sh 'mvn clean package -DskipTests=true'
                // mvn clean package -DskipTests=true
                // mvn clean package -Dmaven.test.skip=true
                archive 'target/*.jar'
            }
        }
        // stage ('Unit Tests'){
        //     steps {
        //         echo "****************** Performing Unit tests for ${env.APPLICATION_NAME} Application ******************"
        //         sh 'mvn test'
        //     }
        // }

        stage ('SonarQube'){
            steps {
                sh """
                echo "Starting Sonar Scan"
                mvn sonar:sonar \
                    -Dsonar.projectKey=i27-eureka \
                    -Dsonar.host.url=http://34.41.182.12:9000 \
                    -Dsonar.login=sqa_577b1c8f0339303219f309fb46bb5f730ce1cf65
                """
            }
        }
    }
}