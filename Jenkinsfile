pipeline {
    agent { label 'Jenkins_Agent' }
    
    environment {
        DOCKER_IMAGE = 'sandeep2862/django-fbv-todo'
        DOCKER_TAG   = "${BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/sandeepaksm/Django-FBV-DRF-TodoApp.git'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'SONAR_AUTH_TOKEN', variable: 'SONAR_TOKEN')]) {
                    sh '''
                        /opt/sonar-scanner/bin/sonar-scanner \
                            -Dsonar.projectKey=django-fbv-todo \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=http://13.54.11.210:9000 \
                            -Dsonar.login=${SONAR_TOKEN}
                    '''
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                sh 'docker build --no-cache -t ${DOCKER_IMAGE}:${DOCKER_TAG} .'
            }
        }
        
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub-cred',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin
                        docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                    '''
                }
            }
        }
        
        stage('Update K8s Manifest') {
            steps {
                withCredentials([string(credentialsId: 'GITHUB_TOKEN', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        git config user.email "sandeepaksm@gmail.com"
                        git config user.name "sandeepaksm"
                        sed -i "s|sandeep2862/django-fbv-todo:.*|sandeep2862/django-fbv-todo:${DOCKER_TAG}|g" k8s/deployment.yaml
                        git add k8s/deployment.yaml
                        git diff --staged --quiet || git commit -m "Update image to version ${DOCKER_TAG}"
                        git push https://${GITHUB_TOKEN}@github.com/sandeepaksm/Django-FBV-DRF-TodoApp.git HEAD:main
                    '''
                }
            }
        }
    }
    
    post {
        always {
            sh 'docker image prune -f || true'
        }
    }
}
