pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
        NAME = 'netflix'
        VERSION = "${env.BUILD_ID}-${env.GIT_COMMIT}"
        IMAGE_REPO = 'chinmayapradhan'
        GITHUB_TOKEN = credentials('github-token')
    }

    stages {
        stage('Checkout from Git') {
            steps {
                script {
                    echo "Clone the repo.."
                    git branch: 'main', url: 'https://github.com/chinmaya10000/DevSecOps-Project.git'
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                script {
                    echo "Run SonarQube Scanner to analyze the code.."
                    withSonarQubeEnv('sonar-server') {
                        sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                        -Dsonar.projectKey=Netflix'''
                    }
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                script {
                    sh "npm install"
                }
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                script {
                    echo "Running OWASP Dependency-Check..."
                    dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                }
            }
        }
        stage('Build and push Image') {
            steps {
                script {
                    echo "build and push the docker image.."
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-repo', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh "docker build --build-arg TMDB_V3_API_KEY=0ab1a657dfc203653a39a9dd4254b6e8 -t ${IMAGE_REPO}/${NAME}:${VERSION} ."
                        sh "echo $PASS | docker login -u $USER --password-stdin"
                        sh "docker push ${IMAGE_REPO}/${NAME}:${VERSION}"
                    }
                }
            }
        }
        stage("Scan image with trivy") {
            steps {
                script {
                    sh "trivy image ${IMAGE_REPO}/${NAME}:${VERSION}"
                }
            }
        }
        stage('Clone/Pull Repo') {
            steps {
                script {
                    if (fileExists('DevSecOps-Project')) {
                        echo 'Cloned repo already exists - Pulling latest changes'

                        dir("DevSecOps-Project") {
                            sh 'git pull'
                        }
                    }
                    else {
                        echo 'Repo does not exists - Cloning the repo'
                        sh 'git clone -b main https://github.com/chinmaya10000/DevSecOps-Project.git'
                    }
                }
            }
        }
        stage('Update Manifest') {
            steps {
                script {
                    dir("DevSecOps-Project/Kubernetes") {
                        sh "sed -i 's#image: ${IMAGE_REPO}/${NAME}:.*#image: ${IMAGE_REPO}/${NAME}:${VERSION}#g' deployment.yml"
                        sh 'cat deployment.yml'
                    }
                }
            }
        }
        stage('Commit & Push') {
            steps {
                script {
                    dir("DevSecOps-Project/Kubernetes") {
                        sh 'git config --global user.name "chinmaya10000"'
                        sh 'git config --global user.email "chinmayapradhan10000@gmail.com"'
                        sh "git remote set-url origin http://$GITHUB_TOKEN@github.com/chinmaya10000/DevSecOps-Project.git"
                        sh 'git add -A'
                        sh 'git reset -- Kubernetes@tmp/'
                        sh 'git commit -am "Updated image version for Build - $VERSION"'
                        sh 'git push origin HEAD:main'
                    }
                }
            }
        }
    }
}    
