pipeline {
    agent any

    environment {
        IMAGE_NAME = 'devops_pipeline_demo'
        CONTAINER_NAME = 'devops_pipeline_demo_container'
        SERVER_PORT = '8180'
        SERVER_PATH = 'sample.txt'
    }

    stages {
        stage('Build') {
            steps {
                echo "..... Build Phase Started :: Compiling Source Code :: ......"
                dir('java_web_code') {
                    sh 'mvn install'
                }
            }
        }

        stage('Test') {
            steps {
                echo "..... Test Phase Started :: Testing via Automated Scripts :: ......"
                dir('integration-testing') {
                    sh 'mvn clean verify -P integration-test'
                }
            }
        }

        stage('Package') {
            steps {
                echo "..... Integration Phase Started :: Copying Artifacts :: ......"
                sh 'cp java_web_code/target/wildfly-spring-boot-sample-1.0.0.war docker/'
            }
        }

        stage('Docker Build') {
            steps {
                echo "..... Provisioning Phase Started :: Building Docker Container :: ......"
                dir('docker') {
                    sh "docker build -t ${IMAGE_NAME} ."
                }
            }
        }

        stage('Deployment') {
            steps {
                echo "..... Deployment Phase Started :: Running Docker Container :: ......"
                script {
                    // Remove container if it already exists
                    def containerExists = sh(script: "docker ps -aq -f name=${CONTAINER_NAME}", returnStdout: true).trim()
                    if (containerExists) {
                        sh "docker rm -f ${CONTAINER_NAME}"
                    }

                    // Run new container
                    sh "docker run -d -p ${SERVER_PORT}:8080 --name ${CONTAINER_NAME} ${IMAGE_NAME}"
                }
            }
        }
    }

    post {
        success {
            echo "--------------------------------------------------------"
            echo "View App deployed here: http://server-ip:${SERVER_PORT}/${SERVER_PATH}"
            echo "--------------------------------------------------------"
        }
        failure {
            echo "Build failed. Check logs for more details."
        }
    }
}
