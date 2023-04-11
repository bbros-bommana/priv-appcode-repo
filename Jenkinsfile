pipeline {
    agent any
    environment {
        AWS_ACCOUNT_ID="356791834110"
        AWS_DEFAULT_REGION="us-east-2"
        IMAGE_REPO_NAME="gitops-skm-demo"
        IMAGE_TAG="${BUILD_NUMBER}"
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
    }

    stages { 
        stage('Cleanup Workspace'){
            steps {
                script {
                    cleanWs()
                }
            }
        }
        
        stage('Logging into AWS ECR') {
            steps {
                script {

                    sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
                }
                 
            }
        }
        
        stage('Cloning Git') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '', url: 'https://github.com/krrish1110/gitops-demo.git']]])     
            }
        }
  

        stage('Building image') {
            steps{ 
                script {                    
                    dockerImage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('Push docker image to ECR registry') {
            steps{
                script{
                    sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:$IMAGE_TAG"
                    sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('Checkout eks manifest GitHub repo') {
            steps{
                git url: 'https://github.com/krrish1110/gitops-demo-config.git',
                branch: 'master'
            }

        }

        stage('Updating Kubernetes deployment file'){
            steps {
                sh "cat deployment.yml"
                sh "sed -i 's/${IMAGE_REPO_NAME}.*/${IMAGE_REPO_NAME}:${IMAGE_TAG}/g' deployment.yml"
                sh "cat deployment.yml"
            }
        }

        stage('Push the changed deployment file to Git'){
            steps {
                script{
                    sh """
                    git config --global user.name "krrish1110"
                    git config --global user.email "krrish.kj1110@gmail.com"
                    git add deployment.yml
                    git commit -m 'Updated the deployment file' """
                    withCredentials([usernamePassword(credentialsId: 'github', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        sh('git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/krrish1110/gitops-demo-config.git HEAD:master')
                    }
                }
            }
        }


    }

}
