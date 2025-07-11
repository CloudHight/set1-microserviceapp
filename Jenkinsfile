pipeline {
    agent any
    environment {
        APP_REPO_NAME = "set1-microserviceapp"
        IMAGE_NAME = "franklinonyia/productcatalogservice"
        BUILD_TAG = "${BUILD_NUMBER}"
        DEPLOYMENT_MANIFEST = "deployment-service.yml"
        GIT_REPO_URL = "https://github.com/CloudHight/set1-microserviceapp.git"
        STAGE_BRANCH = "stage"
        MAIN_BRANCH = "main"
        SLACK_CHANNEL = "#16th-june-ecommerce-project-using-kops-eu-team1"
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
                    withCredentials([usernamePassword(credentialsId: 'git-cred', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_TOKEN')]) {
                        sh """
                            rm -rf ${APP_REPO_NAME} || true
                            git clone ${GIT_REPO_URL}
                            cd ${APP_REPO_NAME}
                            git config --global user.email "jenkins@set30.space"
                            git config --global user.name "Jenkins CI"
                            git checkout ${STAGE_BRANCH}
                            git pull origin ${STAGE_BRANCH} --rebase
                            sed -i 's|image: ${IMAGE_NAME}:.*|image: ${IMAGE_NAME}:${BUILD_TAG}|' ${DEPLOYMENT_MANIFEST}
                            git add ${DEPLOYMENT_MANIFEST}
                            if git diff --cached --quiet; then
                                echo "No changes detected, skipping commit in stage branch."
                            else
                                git commit -m "Update image tag to ${IMAGE_NAME}:${BUILD_TAG} in stage branch"
                                git push https://${GIT_USERNAME}:${GIT_TOKEN}@${GIT_REPO_URL.replace('https://', '')} ${STAGE_BRANCH}
                            fi
                        """
                    }
                }
            }
        }

        stage('Manual Approval for Main Branch Update') {
            steps {
                script {
                    slackSend(channel: SLACK_CHANNEL, message: " *Manual Approval Required for productcatalog pipeline!* Please approve deployment to the *main* branch.")
                }
                input message: "Approve updating deployment-service.yml in the main branch?"
            }
        }

        stage('Update Deployment Manifest in Main Branch') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'git-cred', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_TOKEN')]) {
                        sh """
                            cd ${APP_REPO_NAME}
                            git checkout ${MAIN_BRANCH}
                            git pull origin ${MAIN_BRANCH} --rebase
                            git config --global user.email "jenkins@set30.space"
                            git config --global user.name "Jenkins CI"
                            sed -i 's|image: ${IMAGE_NAME}:.*|image: ${IMAGE_NAME}:${BUILD_TAG}|' ${DEPLOYMENT_MANIFEST}
                            git add ${DEPLOYMENT_MANIFEST}
                            if git diff --cached --quiet; then
                                echo "No changes detected, skipping commit in main branch."
                            else
                                git commit -m "Update image tag to ${IMAGE_NAME}:${BUILD_TAG} in main branch"
                                git push https://${GIT_USERNAME}:${GIT_TOKEN}@${GIT_REPO_URL.replace('https://', '')} ${MAIN_BRANCH}
                            fi
                        """
                    }
                }
            }
        }

        stage('Clean up disk') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker rmi ${IMAGE_NAME}:${BUILD_TAG}"
                    }
                }
            }
        }
    }

    post {
        success {
            script {
                slackSend(channel: SLACK_CHANNEL, message: "✅ *Build Succeeded!* Jenkins has successfully completed the productcatalog pipeline. 🎉")
            }
        }
        failure {
            script {
                slackSend(channel: SLACK_CHANNEL, message: "❌ *Build Failed! for productcatalog pipeline* Please check the Jenkins logs for details. 🔴")
            }
        }
    }
}
