pipeline {
    agent any

    environment {
        DOCKER_USERNAME = 'anandhakumarg'
        DOCKER_IMAGE = 'anand-spring-boot-app'
        DOCKER_TAG = 'v1'
        REGISTRY_CREDENTIALS = credentials('docker-password')
        GIT_USER_NAME = 'ANANDHAKUMAR18'
        GIT_REPO_NAME = 'Spring-Boot-Application-CICD.git'
        DOCKER_REGISTRY = 'docker.io'
    }

    tools {
        maven 'Maven'
    }

    

    stages {

       stage('User') {
           steps {
               script {
                   sh 'whoami'
               }
           }
       }

       stage('Code Checkout') {
        steps {
            git branch: 'main', url: 'https://github.com/ANANDHAKUMAR18/Spring-Boot-Application-CICD.git' 
        }
       }

       stage('Build and Test') {
        steps {
            sh 'cd java-maven-sonar-argocd-helm-k8s/spring-boot-app && mvn clean compile package'
        }
       }

       stage('Build and Push Docker Image') {
        steps {
            withCredentials([string(credentialsId: 'docker-password', variable: 'DOCKER_PASSWORD')]) {
                script {
                    sh "docker login -u ${DOCKER_USERNAME} -p ${env.DOCKER_PASSWORD} ${DOCKER_REGISTRY}"
                    sh "cd java-maven-sonar-argocd-helm-k8s/spring-boot-app &&  docker build -t ${DOCKER_USERNAME}/${DOCKER_IMAGE}:${DOCKER_TAG} ."
                    sh " docker push ${DOCKER_USERNAME}/${DOCKER_IMAGE}:${DOCKER_TAG}"
                   
                }
            }
        }
       }

       stage('Update Deployment File') {
        steps {
            withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                script {
                    
                        sh """
                        git config user.email "anandh@google.com"
                        git config user.name "AnandhakumarG"
                        sed -i "s/tag/ ${DOCKER_TAG}/ " java-maven-sonar-argocd-helm-k8s/spring-boot-app-manifests/deployment.yml
                        
                        git add .
                        git commit -m "Update Deployment Manifest File to Version ${DOCKER_TAG}"
                        git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main

                        """
                    }   
                }
            }
        }
       stage('Deploy Argocd Application') {
           steps {
               scripts {
                   sh 'cd java-maven-sonar-argocd-helm-k8s/spring-boot-app-manifests/'
                   sh 'kubectl apply -f application.yml'
               }
           }
       }
    }

}
