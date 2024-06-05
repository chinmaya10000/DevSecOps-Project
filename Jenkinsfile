pipeline {

    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }

    environment {
        SCANNER_HOME=tool 'sonar-scanner'
        NAME = 'netflix'
        IMAGE_VERSION = "${env.BUILD_NUMBER}"
        IMAGE_REPO = 'chinmayapradhan'
    }

    stages {
        stage("Unit Test") {
            steps {
                script {
                    echo 'Implement unit tests if applicable.'
                    echo 'This stage is a sample placeholder'
                }
            }
        }
        stage("SonarQube Analysis") {
            steps {
                script {
                    withSonarQubeEnv('sonar-server') {
                        sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                        -Dsonar.projectKey=Netflix'''
                    }
                }
            }
        }
        stage("Install Dependencies") {
            steps {
                script {
                    sh 'npm install'
                }
            }
        }
        stage("OWASP FS SCAN") {
            steps {
                script {
                    dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                }
            }
        }
        stage("Build and Push Image") {
            steps {
                script {
                    echo "Build and push image"
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-repo', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh "docker build -t ${IMAGE_REPO}/${NAME}:${IMAGE_VERSION} ."
                        sh "echo $PASS | docker login -u $USER --password-stdin"
                        sh "docker push ${IMAGE_REPO}/${NAME}:${IMAGE_VERSION}"
                    }
                }
            }
        }
        stage("TRIVY scan") {
            steps {
                script {
                    sh "trivy image > ${IMAGE_REPO}/${NAME}:${IMAGE_VERSION}"
                }
            }
        }
    }
}