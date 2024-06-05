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
                        sh "docker build --build-arg TMDB_V3_API_KEY=0ab1a657dfc203653a39a9dd4254b6e8 -t ${IMAGE_REPO}/${NAME}:${IMAGE_VERSION} ."
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
        stage("Push Deployment YAML to GitHub") {
            steps {
                script {
                    withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                        sh '''
                          git config --global user.name "chinmaya1000"
                          git config --global user.email "chinmayapradhan10000@gmail.com"
                          git remote set-url origin https://${GITHUB_TOKEN}@github.com/chinmaya10000/DevSecOps-Project.git
                          sed -i "s#chinmayapradhan.*${IMAGE_REPO}/${NAME}:${IMAGE_VERSION}#g" Kubernetes/deployment.yml
                          git add Kubernetes/deployment.yml
                          git commit -m "Updated image version for Build - $IMAGE_VERSION"
                          git push origin HEAD:main
                        '''
                    }
                }
            }
        }
    }
}