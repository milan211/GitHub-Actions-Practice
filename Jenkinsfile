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
        POM_VERSION = readMavenPom().getVersion() 
        POM_PACKAGING = readMavenPom().getPackaging()
        DOCKER_HUB = "docker.io/i27devopsb5"
        DOCKER_CREDS = credentials('dockerhub_creds') // username and password
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
                // COde Quality needs to be implemented in this stage
                // Before we execute or write the code, make sure sonarqube-scanner plugin is installed.
                // sonar details are ben configured in the Manage Jenkins > system 
                echo "****************** Starting Sonar Scans with Quality Gates ******************"
                withSonarQubeEnv('SonarQube') { // SonarQube is the name we configured in Manage Jenkins > System > Sonarqube , it should match exactly, 
                    sh """
                        mvn sonar:sonar \
                            -Dsonar.projectKey=i27-eureka \
                            -Dsonar.host.url=http://35.188.56.142:9000 \
                            -Dsonar.login=sqa_577b1c8f0339303219f309fb46bb5f730ce1cf65
                    """
                }
                timeout (time: 2, unit: 'MINUTES') { //NANOSECONDS, SECONDS, MINUTES, HOURS, DAYS
                     waitForQualityGate abortPipeline: true
                }
   
            }
        }
        stage ('BuildFormat') {
            steps {
                script { 
                    // Existing : i27-eureka-0.0.1-SNAPSHOT.jar
                    // Destination: i27-eureka-buildnumber-branchname.packagin
                  sh """
                    echo "Testing JAR Source: i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING}"
                    echo "Testing JAR Destination Format: i27-${env.APPLICATION_NAME}-${currentBuild.number}-${BRANCH_NAME}.${env.POM_PACKAGING}"
                  """

                }
            }
        }
        stage ('Docker Build Push') {
            steps {
                script {
                    echo "****************** Building Docker image ******************"
                    sh "cp ${WORKSPACE}/target/i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} ./.cicd"
                    sh "docker build --no-cache --build-arg JAR_SOURCE=i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} -t ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT} ./.cicd/"
                    echo "****************** Login to Docker Registry ******************"
                    sh "docker login -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}"
                    echo "****************** Push Image to Docker Registry ******************"
                    sh "docker push ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}"

                }
            }
        }
        stage ('Deploy to Dev env'){
            steps {
                echo "****************** Deploying to Dev Env ******************"
                withCredentials([usernamePassword(credentialsId: 'john_docker_vm_passwd', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                        // some block
                        // we will communicate to the server
                        script {
                            // Command/syntax to use sshpass
                            //$ sshpass -p !4u2tryhack ssh -o StrictHostKeyChecking=no username@host.example.com
                            // Create container 
                            sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no '$USERNAME'@$dev_ip \"docker container run -dit -p 8761:8761 --name eureka-dev ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT} \""
                        }
                }
            }
        }
    }
}