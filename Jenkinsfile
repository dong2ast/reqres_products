pipeline {
    agent any

    environment {
        REGISTRY = 'user02.azurecr.io'
        IMAGE_NAME = 'product'
        AKS_CLUSTER = 'user02-aks'
        RESOURCE_GROUP = 'user02-rsrcgrp'
        AKS_NAMESPACE = 'default'
        AZURE_CREDENTIALS_ID = 'Azure-Cred'
        TENANT_ID = 'f46af6a3-e73f-4ab2-a1f7-f33919eda5ac' // Service Principal 등록 후 생성된 ID
        GIT_USER_NAME = 'dong2ast'
        GIT_USER_EMAIL = 'kdh01132006@naver.com'
        GITHUB_CREDENTIALS_ID = 'Github-Cred'
        GITHUB_REPO = 'github.com/dong2ast/reqres_products.git'
        GITHUB_BRANCH = 'master' // 업로드할 브랜치
    }
 
    stages {
        stage('Clone Repository') {
            when {
                expression {
                    currentBuild.result != 'NOT_BUILT'
                }
            }
            steps {
                checkout scm
            }
        }
        
        stage('Maven Build') {
            when {
                expression {
                    currentBuild.result != 'NOT_BUILT'
                }
            }
            steps {
                withMaven(maven: 'Maven') {
                    sh 'mvn package -DskipTests'
                }
            }
        }
        
        stage('Docker Build') {
            when {
                expression {
                    currentBuild.result != 'NOT_BUILT'
                }
            }
            steps {
                script {
                    image = docker.build("${REGISTRY}/${IMAGE_NAME}:v${env.BUILD_NUMBER}")
                }
            }
        }
        
        stage('Push to ACR') {
            when {
                expression {
                    currentBuild.result != 'NOT_BUILT'
                }
            }
            steps {
                script {
                    sh "az acr login --name ${REGISTRY.split('\\.')[0]}"
                    sh "docker push ${REGISTRY}/${IMAGE_NAME}:v${env.BUILD_NUMBER}"
            
                }
            }
        }

    
        
        stage('CleanUp Images') {
            when {
                expression {
                    currentBuild.result != 'NOT_BUILT'
                }
            }
            steps {
                sh """
                docker rmi ${REGISTRY}/${IMAGE_NAME}:v$BUILD_NUMBER
                """
            }
        }
        
        stage('Update deploy.yaml') {
            when {
                expression {
                    currentBuild.result != 'NOT_BUILT'
                }
            }
            steps {
                script {
                    sh """
                    sed -i 's|image: \"${REGISTRY}/${IMAGE_NAME}:.*\"|image: \"${REGISTRY}/${IMAGE_NAME}:v${env.BUILD_ID}\"|' kubernetes/deploy.yaml
                    cat kubernetes/deploy.yaml
                    """
                }
            }
        }
        
        stage('Commit and Push to GitHub') {
            when {
                expression {
                    currentBuild.result != 'NOT_BUILT'
                }
            }
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: GITHUB_CREDENTIALS_ID, usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                        sh """
                            git config --global user.email "kdh01132006@naver.com"
                            git config --global user.name "dong2ast"
                            git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@${GITHUB_REPO} repo
                            cp kubernetes/deploy.yaml repo/kubernetes/deploy.yaml
                            cd repo
                            git add kubernetes/deploy.yaml
                            git commit -m "Update deploy.yaml with build ${env.BUILD_NUMBER}"
                            git push origin ${GITHUB_BRANCH}
                            cd ..
                            rm -rf repo
                        """
                    }
                }
            }
        } 
    }
}
