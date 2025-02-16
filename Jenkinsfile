pipeline {
    agent any
    environment {
        IMAGE_NAME = "bwhizzy25/adservice"
        BUILD_TAG = "${BUILD_NUMBER}"
        DEPLOYMENT_MANIFEST = "deployment-service.yml"
        GIT_REPO_URL = "https://github.com/CloudHight/set1-microserviceapp.git"
        STAGE_BRANCH = "stage"
        MAIN_BRANCH = "main"
    }
    stages {
        stage('Build & Tag Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t ${IMAGE_NAME}:${BUILD_TAG} ."
                    }
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push ${IMAGE_NAME}:${BUILD_TAG}"
                    }
                }
            }
        }
        stage('Update Deployment Manifest in Stage Branch') {
            steps {
                script {
                    sh """
                        git clone ${GIT_REPO_URL}
                        cd set1-microserviceapp
                        git checkout ${STAGE_BRANCH}
                        sed -i 's|image: ${IMAGE_NAME}:.*|image: ${IMAGE_NAME}:${BUILD_TAG}|' ${DEPLOYMENT_MANIFEST}
                        git add ${DEPLOYMENT_MANIFEST}
                        git commit -m "Update image tag to ${IMAGE_NAME}:${BUILD_TAG} in stage branch"
                        git push origin ${STAGE_BRANCH}
                    """
                }
            }
        }
        stage('Manual Approval for Main Branch Update') {
            steps {
                input message: "Approve updating deployment-service.yml in the main branch?"
            }
        }
        stage('Update Deployment Manifest in Main Branch') {
            steps {
                script {
                    sh """
                        cd set1-microserviceapp
                        git checkout ${MAIN_BRANCH}
                        sed -i 's|image: ${IMAGE_NAME}:.*|image: ${IMAGE_NAME}:${BUILD_TAG}|' ${DEPLOYMENT_MANIFEST}
                        git add ${DEPLOYMENT_MANIFEST}
                        git commit -m "Update image tag to ${IMAGE_NAME}:${BUILD_TAG} in main branch"
                        git push origin ${MAIN_BRANCH}
                    """
                }
            }
        }
    }
}
