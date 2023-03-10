properties([pipelineTriggers([githubPush()])])

// Comment 1 added to perform a commit
// Comment 2 added to perform a commit

pipeline {
    agent any
    options {
        buildDiscarder(logRotator(numToKeepStr: '5', daysToKeepStr: '5'))
    }
    environment {
        WORKSPACE_ = "/volume/jenkins/workspace/"
        IMAGE_NAME = "mikhailklimov/pacman-demo-test"
        REPOSITORY_ = "mikhailklimov1/pacman-demo-test"
        BRANCH_ = "main"
        GITHUB_CREDENTIALS = credentials('github_creds')
        REGISTRY_= "docker.io"
        DOCKERHUB_CREDENTIALS = credentials('dockerhub_creds')
    }
    stages {
// This stage is needed to make a checkout manually 
//        stage("Git checkout") {
//            steps {
//                ws("${WORKSPACE_}") {
//                git credentialsId: 'GITHUB_CREDENTIALS', url: "https://github.com/${REPOSITORY_}", branch: "${BRANCH_}" 
//                // Check if it possible to add var instead of specific repo name here ^
//                sh "echo Git Checkout Completed"
//                }
//            }
//        }
        stage('Checkout SCM') {
            steps {
                ws("${WORKSPACE_}") {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: "${BRANCH_}"]],
                        userRemoteConfigs: [[
                            url: "https://github.com/${REPOSITORY_}",
                            credentialsId: 'GITHUB_CREDENTIALS',
                        ]]
                    ])
                }
            }
        }
        stage ('Build image') {
            steps {
                ws("${WORKSPACE_}") {    
                    sh "podman build -t ${IMAGE_NAME}:${GIT_COMMIT} ."
                    sh "echo Build Image Completed"
                }			
            }
        }
        stage('Login to Docker Hub') {
            steps {
                ws("${WORKSPACE_}") {
                    sh "echo ${DOCKERHUB_CREDENTIALS_PSW} | podman login ${REGISTRY_} -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin"
                    sh "echo Login Completed" 
                }
            }           
        }
        stage('Push image to Docker Hub') {
            steps {
                ws("${WORKSPACE_}") {
                    sh "podman push ${IMAGE_NAME}:${GIT_COMMIT}"
                    sh "echo Push Image Completed"
                }
            }
        }
    }
    post {
        always {
            ws("${WORKSPACE_}") {
                sh "podman rmi ${IMAGE_NAME}:${GIT_COMMIT}"
                sh "podman logout ${REGISTRY_}"
            }
        }                
    }     
}
