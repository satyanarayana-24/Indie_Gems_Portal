pipeline {
    agent any

    environment {
        // WORK_DIR = "/var/lib/jenkins/workspace/Game"
        IMAGE_NAME = "indie-gems"
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKERHUB_USER = "9397054542"
        DOCKER_CREDS = "dockerCred"
        AWS_REGION = "ap-south-1"
        EKS_CLUSTER = "myclusterr"
        KUBECONFIG = "/var/lib/jenkins/.kube/config"
        NOTIFY_EMAIL = "satyanarayana.gidituri666@gmail.com"
    }

    stages {

        stage('Checkout Code') {
            steps {
                script {
                    try {
                        dir("${WORK_DIR}") {
                            git branch: 'main',
                                url: 'https://github.com/satyanarayana-24/Indie_Gems_Portal.git'
                        }

                        sendMail("Checkout Code", "SUCCESS")

                    } catch (e) {
                        sendMail("Checkout Code", "FAILURE")
                        error("Checkout failed")
                    }
                }
            }
        }
    //nexus starting
         stage('COMPILE') {
            steps {
              sh 'mvn clean package -X'
            }
        } 
        stage('JENKINS TO NEXUS') {
            steps {
              withMaven(globalMavenSettingsConfig: 'settings.xml', jdk: 'jkd17', maven: 'maven3', traceability: true) {
             sh 'mvn deploy'
             }
            }
        }         
    //nexus ending    

        stage('Build Docker Image') {
            steps {
                script {
                    try {
                        dir("${WORK_DIR}") {
                            sh '''
                                docker rmi -f ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} || true
                                docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} .
                            '''
                        }

                        sendMail("Build Docker Image", "SUCCESS")

                    } catch (e) {
                        sendMail("Build Docker Image", "FAILURE")
                        error("Build failed")
                    }
                }
            }
        }

        stage('DockerHub Login') {
            steps {
                script {
                    try {
                        withCredentials([usernamePassword(
                            credentialsId: "${DOCKER_CREDS}",
                            usernameVariable: 'DOCKER_USER',
                            passwordVariable: 'DOCKER_PASS'
                        )]) {
                            sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                        }

                        sendMail("DockerHub Login", "SUCCESS")

                    } catch (e) {
                        sendMail("DockerHub Login", "FAILURE")
                        error("Docker login failed")
                    }
                }
            }
        }

        stage('Push Image') {
            steps {
                script {
                    try {
                        sh "docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"

                        sendMail("Push Image", "SUCCESS")

                    } catch (e) {
                        sendMail("Push Image", "FAILURE")
                        error("Push failed")
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    try {
                        dir("${WORK_DIR}") {
                            sh '''
                                sed -i "s|image:.*|image: $DOCKERHUB_USER/$IMAGE_NAME:$IMAGE_TAG|" k8s/deployment.yml
                                aws eks --region $AWS_REGION update-kubeconfig --name $EKS_CLUSTER
                                kubectl apply -f k8s/deployment.yml
                                kubectl apply -f k8s/service.yml
                            '''
                        }

                        sendMail("Deploy to Kubernetes", "SUCCESS")

                    } catch (e) {
                        sendMail("Deploy to Kubernetes", "FAILURE")
                        error("Deployment failed")
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline completed: ${currentBuild.currentResult}"
        }
    }
}

def sendMail(stageName, status) {
    emailext(
        subject: "${status}: ${stageName} - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body: """
            <h3>Stage: ${stageName}</h3>
            <p><b>Status:</b> ${status}</p>
            <p><b>Job:</b> ${env.JOB_NAME}</p>
            <p><b>Build:</b> ${env.BUILD_NUMBER}</p>
            <p><b>URL:</b> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
        """,
        to: "${env.NOTIFY_EMAIL}",
        mimeType: 'text/html'
    )
}
// below is for sucess fail abort
// pipeline {
//     agent any

//     environment {
//         WORK_DIR = "/var/lib/jenkins/workspace/Game"
//         IMAGE_NAME = "indie-gems"
//         IMAGE_TAG = "${BUILD_NUMBER}"
//         CONTAINER_NAME = "indie-gems-container"
//         PORT = "9676"
//         DOCKERHUB_USER = "9397054542"
//         DOCKER_CREDS = "dockerCred"
//         CONTAINER_PORT = "80"
//         AWS_REGION = "ap-south-1"
//         EKS_CLUSTER = "myclusterr"
//         KUBECONFIG = "/var/lib/jenkins/.kube/config"
//         NOTIFY_EMAIL = "satyanarayana.gidituri666@gmail.com"
//     }

//     stages {

//         stage('Checkout Code') {
//             steps {
//                 dir("${WORK_DIR}") {
//                     git branch: 'main',
//                         url: 'https://github.com/satyanarayana-24/Indie_Gems_Portal.git'
//                 }
//             }
//         }

//         stage('Build Docker Image') {
//             steps {
//                 dir("${WORK_DIR}") {
//                     sh '''
//                         docker rmi -f ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} || true
//                         docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} .
//                     '''
//                 }
//             }
//         }

//         stage('DockerHub Login') {
//             steps {
//                 withCredentials([usernamePassword(
//                     credentialsId: "${DOCKER_CREDS}",
//                     usernameVariable: 'DOCKER_USER',
//                     passwordVariable: 'DOCKER_PASS'
//                 )]) {
//                     sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
//                 }
//             }
//         }

//         stage('Push Image to DockerHub') {
//             steps {
//                 sh "docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
//             }
//         }

//         stage('Update K8s Image') {
//             steps {
//                 dir("${WORK_DIR}") {
//                     sh '''
//                         sed -i "s|image:.*|image: $DOCKERHUB_USER/$IMAGE_NAME:$IMAGE_TAG|" k8s/deployment.yml
//                     '''
//                 }
//             }
//         }

//         stage('Configure EKS Access') {
//             steps {
//                 sh '''
//                     export PATH=$PATH:/usr/local/bin
//                     aws eks --region $AWS_REGION update-kubeconfig --name $EKS_CLUSTER
//                     kubectl config current-context
//                 '''
//             }
//         }

//         stage('Deploy to Kubernetes') {
//             steps {
//                 dir("${WORK_DIR}") {
//                     sh '''
//                         kubectl apply -f k8s/deployment.yml
//                         kubectl apply -f k8s/service.yml
//                     '''
//                 }
//             }
//         }

//         stage('Verify Deployment') {
//             steps {
//                 sh '''
//                     kubectl rollout status deployment python-devops-app || true
//                     kubectl get pods -o wide
//                     kubectl get svc
//                 '''
//             }
//         }
//     }

//     post {

//         always {
//             echo "Pipeline finished with status: ${currentBuild.currentResult}"
//         }

//         success {
//             emailext(
//                 subject: " SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
//                 body: """
//                     <h2 style="color:green;">Build & Deployment Successful</h2>
//                     <p><b>Job:</b> ${env.JOB_NAME}</p>
//                     <p><b>Build Number:</b> ${env.BUILD_NUMBER}</p>
//                     <p><b>Status:</b> ${currentBuild.currentResult}</p>
//                     <p><b>Docker Image:</b> ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}</p>
//                     <p><b>URL:</b> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
//                 """,
//                 to: "${NOTIFY_EMAIL}",
//                 mimeType: 'text/html'
//             )
//         }

//         failure {
//             emailext(
//                 subject: " FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
//                 body: """
//                     <h2 style="color:red;">Build or Deployment Failed</h2>
//                     <p><b>Job:</b> ${env.JOB_NAME}</p>
//                     <p><b>Build Number:</b> ${env.BUILD_NUMBER}</p>
//                     <p><b>Status:</b> ${currentBuild.currentResult}</p>
//                     <p><b>Check Logs:</b> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
//                 """,
//                 to: "${NOTIFY_EMAIL}",
//                 mimeType: 'text/html'
//             )
//         }

//         unstable {
//             emailext(
//                 subject: " UNSTABLE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
//                 body: """
//                     <h3>Build is Unstable</h3>
//                     <p>Check details: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
//                 """,
//                 to: "${NOTIFY_EMAIL}",
//                 mimeType: 'text/html'
//             )
//         }

//         aborted {
//             emailext(
//                 subject: " ABORTED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
//                 body: """
//                     <h3>Build was Aborted</h3>
//                     <p>Check details: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
//                 """,
//                 to: "${NOTIFY_EMAIL}",
//                 mimeType: 'text/html'
//             )
//         }
//     }
// }
//  above is for sucess fail abort unstable
// commented for get all stages mail this give mail for only one stage
// pipeline {
//     agent any

//     environment {
//         WORK_DIR = "/var/lib/jenkins/workspace/Game"
//         IMAGE_NAME = "indie-gems"
//         IMAGE_TAG = "${BUILD_NUMBER}"
//         CONTAINER_NAME = "indie-gems-container"
//         PORT = "9676"
//         DOCKERHUB_USER = "9397054542"
//         DOCKER_CREDS = "dockerCred"
//         CONTAINER_PORT = "80"
//         AWS_REGION = "ap-south-1"
//         EKS_CLUSTER = "mycluster"
//         KUBECONFIG = "/var/lib/jenkins/.kube/config"
//         NOTIFY_EMAIL = "satyanarayana.gidituri666@gmail.com"
//     }

//     stages {
//         stage('Checkout Code') {
//             steps {
//                 dir("${WORK_DIR}") {
//                     git branch: 'main', url: 'https://github.com/satyanarayana-24/Indie_Gems_Portal.git'
//                 }
//             }
//         }

//         stage('Build Docker Image') {
//             steps {
//                 dir("${WORK_DIR}") {
//                     sh '''
//                         docker rmi -f ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} || true
//                         docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} .
//                     '''
//                 }
//             }
//         }

//         stage('DockerHub Login') {
//             steps {
//                 withCredentials([usernamePassword(
//                     credentialsId: "${DOCKER_CREDS}",
//                     usernameVariable: 'DOCKER_USER',
//                     passwordVariable: 'DOCKER_PASS'
//                 )]) {
//                     sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
//                 }
//             }
//         }

//         stage('Push Image to DockerHub') {
//             steps {
//                 sh "docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
//             }
//         }

//         stage('Update K8s Image') {
//             steps {
//                 sh '''
//                     sed -i "s|image:.*|image: $DOCKERHUB_USER/$IMAGE_NAME:$IMAGE_TAG|" k8s/deployment.yml
//                 '''
//             }
//         }

//         stage('Configure EKS Access') {
//             steps {
//                 sh '''
//                     export PATH=$PATH:/usr/local/bin
//                     aws eks --region $AWS_REGION update-kubeconfig --name $EKS_CLUSTER
//                     /usr/local/bin/kubectl config current-context
//                 '''
//             }
//         }

//         stage('Deploy to Kubernetes') {
//             steps {
//                 sh '''
//                     kubectl apply -f k8s/deployment.yml
//                     kubectl apply -f k8s/service.yml
//                 '''
//             }
//         }

//         stage('Verify Deployment') {
//             steps {
//                 sh '''
//                     kubectl rollout status deployment python-devops-app || true
//                     kubectl get pods -o wide
//                     kubectl get svc
//                 '''
//             }
//         }
//     }

//     post {
//         success {
//             emailext(
//                 subject: "Build & Deploy Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
//                 body: """
//                     <p>Good news!</p>
//                     <p>Build and deployment were successful.</p>
//                     <p><b>Job:</b> ${env.JOB_NAME}</p>
//                     <p><b>Build Number:</b> ${env.BUILD_NUMBER}</p>
//                     <p><b>URL:</b> ${env.BUILD_URL}</p>
//                 """,
//                 to: "${NOTIFY_EMAIL}",
//                 mimeType: 'text/html'
//             )
//         }
//         failure {
//             emailext(
//                 subject: "Build & Deploy Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
//                 body: "Build or deployment failed. Check details at ${env.BUILD_URL}",
//                 to: "${NOTIFY_EMAIL}"
//             )
//         }
//     }
// }
// commeted above for send mail to one stage i want to send for all stages


// starting for k8s automate without email send
// pipeline {
//     agent any

//     environment {
//         WORK_DIR = "/var/lib/jenkins/workspace/Game"
//         IMAGE_NAME = "indie-gems"
//         IMAGE_TAG = "${BUILD_NUMBER}"
//         CONTAINER_NAME = "indie-gems-container"
//         PORT = "9676"
//         DOCKERHUB_USER = "9397054542"
//         DOCKER_CREDS = "dockerCred"
//         CONTAINER_PORT = "80"
//         AWS_REGION = "ap-south-1"
//         EKS_CLUSTER = "mycluster"
//         KUBECONFIG = "/var/lib/jenkins/.kube/config"
//     }

//     stages {
//         stage('Checkout Code') {
//             steps {
//                 dir("${WORK_DIR}") {
//                     git branch: 'main', url: 'https://github.com/satyanarayana-24/Indie_Gems_Portal.git'
//                 }
//             }
//         }

//         stage('Build Docker Image') {
//             steps {
//                 dir("${WORK_DIR}") {
//                     sh '''
//                         docker rmi -f ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} || true
//                         docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} .
//                     '''
//                 }
//             }
//         }

//         stage('DockerHub Login') {
//             steps {
//                 withCredentials([usernamePassword(
//                     credentialsId: "${DOCKER_CREDS}",
//                     usernameVariable: 'DOCKER_USER',
//                     passwordVariable: 'DOCKER_PASS'
//                 )]) {
//                     sh """
//                         echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
//                     """
//                 }
//             }
//         }

//         stage('Push Image to DockerHub') {
//             steps {
//                 sh "docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
//             }
//         }

//         // stage('Run Docker Container') {
//         //     steps {
//         //         sh '''
//         //             docker rm -f ${CONTAINER_NAME} || true
//         //             docker run -d -p ${PORT}:${CONTAINER_PORT} \
//         //             --name ${CONTAINER_NAME} \
//         //             ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
//         //         '''
//         //     }
//         // }

//             stage('Update K8s Image') {
//             steps {
//                 sh '''
//                 sed -i "s|image:.*|image: $DOCKERHUB_USER/$IMAGE_NAME:$IMAGE_TAG|" k8s/deployment.yml
//                 '''
//             }
//         }

//         // stage('Configure EKS Access') {
//         //     steps {
//         //         sh '''
//         //         aws eks --region $AWS_REGION update-kubeconfig --name $EKS_CLUSTER
//         //         kubectl config current-context
//         //         '''
//         //     }
//         // }

// //       stage('Configure EKS Access') {
// //     steps {
// //         sh '''
// //         export PATH=$PATH:/usr/bin
// //         aws eks --region $AWS_REGION update-kubeconfig --name $EKS_CLUSTER
// //         kubectl config current-context
// //         '''
// //     }
// // }
// //         stage('Configure EKS Access') {
// //     steps {
// //         sh '''
// //         aws eks --region $AWS_REGION update-kubeconfig --name $EKS_CLUSTER
// //         /root/bin/kubectl config current-context
// //         '''
// //     }
// // }

//         stage('Configure EKS Access') {
//     steps {
//         sh '''
//         export PATH=$PATH:/usr/local/bin
//         aws eks --region $AWS_REGION update-kubeconfig --name $EKS_CLUSTER
//         /usr/local/bin/kubectl config current-context
//         '''
//     }
// }

//         stage('Deploy to Kubernetes') {
//             steps {
//                 sh '''
//                 kubectl apply -f k8s/deployment.yml
//                 kubectl apply -f k8s/service.yml
//                 '''
//             }
//         }

//         stage('Verify Deployment') {
//             steps {
//                 sh '''
//                 kubectl rollout status deployment python-devops-app || true
//                 kubectl get pods -o wide
//                 kubectl get svc
//                 '''
//             }
//         }
//     }
// }
// upto here this is my k8s automate working fine code  /////////////

// working for automate without k8s
// pipeline {
//     agent any

//     environment {
//         WORK_DIR = "/var/lib/jenkins/workspace/Game"
//         IMAGE_NAME = "indie-gems"
//         IMAGE_TAG = "${BUILD_NUMBER}"
//         CONTAINER_NAME = "indie-gems-container"
//         PORT = "9676"
//         DOCKERHUB_USER = "9397054542"
//         DOCKER_CREDS = "dockerCred"
//         CONTAINER_PORT = "80"
//     }

//     stages {
//         stage('Checkout Code') {
//             steps {
//                 dir("${WORK_DIR}") {
//                     git branch: 'main', url: 'https://github.com/satyanarayana-24/Indie_Gems_Portal.git'
//                 }
//             }
//         }

//         stage('Build Docker Image') {
//             steps {
//                 dir("${WORK_DIR}") {
//                     sh '''
//                         docker rmi -f ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} || true
//                         docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} .
//                     '''
//                 }
//             }
//         }

//         stage('DockerHub Login') {
//             steps {
//                 withCredentials([usernamePassword(
//                     credentialsId: "${DOCKER_CREDS}",
//                     usernameVariable: 'DOCKER_USER',
//                     passwordVariable: 'DOCKER_PASS'
//                 )]) {
//                     sh """
//                         echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
//                     """
//                 }
//             }
//         }

//         stage('Push Image to DockerHub') {
//             steps {
//                 sh "docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
//             }
//         }

//         stage('Run Docker Container') {
//             steps {
//                 sh '''
//                     docker rm -f ${CONTAINER_NAME} || true
//                     docker run -d -p ${PORT}:${CONTAINER_PORT} \
//                     --name ${CONTAINER_NAME} \
//                     ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
//                 '''
//             }
//         }
//     }
// }
// commented for gpt code
// pipeline {
//     agent any

//     environment {
//         WORK_DIR = "/var/lib/jenkins/workspace/Game"
//         IMAGE_NAME = "indie-gems"
//         IMAGE_TAG       = "${BUILD_NUMBER}"
//         CONTAINER_NAME = "indie-gems-container"
//         PORT = "9676"   // External port for app
//         DOCKERHUB_USER  = "9397054542"
//         DOCKER_CREDS    = "dockerCred"
//         CONTAINER_PORT  = "80"
//     }

//     stages {
//         stage('Checkout Code') {
//             steps {
//                 dir("${WORK_DIR}") {
//                     git branch: 'main', url: 'https://github.com/satyanarayana-24/Indie_Gems_Portal.git'
//                 }
//             }
//         }
//         stage('Build Docker Image') {
//     steps {
//         dir("${WORK_DIR}") {
//             sh '''
//                 # Remove old image if exists
//                 docker rmi -f ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} || true

//                 # Build new image
//                 docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} .
//             '''
//         }
//     }
// }


//          stage('DockerHub Login') {
//             steps {
//                 withCredentials([usernamePassword(
//                     credentialsId: "${DOCKER_CREDS}",
//                     usernameVariable: 'DOCKER_USER',
//                     passwordVariable: 'DOCKER_PASS'
//                 )]) {

//                     sh """
//                     echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
//                     """
//                 }
//             }
//         }

//          stage('Push Image to DockerHub') {
//             steps {
//                 sh """
//                 docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
//                 """
//             }
//         }


//         stage('Run Docker Container') {
//     steps {
//         dir("${WORK_DIR}") {
//             sh '''
//                 docker rm -f ${CONTAINER_NAME} || true

//                 docker run -d \
//                 -p ${PORT}:${CONTAINER_PORT} \
//                 --name ${CONTAINER_NAME} \
//                 ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
//             '''
//         }
//     }


//         // stage('Run Docker Container') {
//         //     steps {
//         //         dir("${WORK_DIR}") {
//         //             sh '''
//         //                 docker rm -f ${CONTAINER_NAME} || true
//         //                 // docker run -d --name ${CONTAINER_NAME} -p ${PORT}:${CONTAINER_PORT} \ ${IMAGE_NAME}:${IMAGE_TAG}
//         //                   docker run -d \
//         //         -p ${HOST_PORT}:${CONTAINER_PORT} \
//         //         --name ${CONTAINER_NAME} \
//         //         ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
//         //             '''
//         //         }
//         //     }
    

    
//     }
// }


//original and docker one is below  ====================

// pipeline {
//     agent any

//     environment {
//         WORK_DIR = "/var/lib/jenkins/workspace/Game"
//         IMAGE_NAME = "indie-gems"
//         IMAGE_TAG       = "${BUILD_NUMBER}"
//         CONTAINER_NAME = "indie-gems-container"
//         PORT = "9676"   // External port for app
//         DOCKERHUB_USER  = "9397054542"
//         DOCKER_CREDS    = "dockerCred"
//         CONTAINER_PORT  = "80"
//     }

//     stages {
//         stage('Checkout Code') {
//             steps {
//                 dir("${WORK_DIR}") {
//                     git branch: 'main', url: 'https://github.com/satyanarayana-24/Indie_Gems_Portal.git'
//                 }
//             }
//         }
// //         stage('Install Docker') {
// //     steps {
// //         sh '''
// //         # Check if docker exists
// //         if ! command -v docker &> /dev/null
// //         then
// //             echo "Docker not found. Installing Docker..."

// //             sudo apt update
// //             sudo apt install docker.io -y

// //             sudo systemctl start docker
// //             sudo systemctl enable docker

// //             // sudo usermod -aG docker jenkins
// //         else
// //             echo "Docker already installed"
// //         fi

// //         docker --version
// //         '''
// //     }
// // }
//         // stage('Build Docker Image') {
//         //     steps {
//         //         dir("${WORK_DIR}") {
//         //             sh '''
//         //                 // docker rmi -f ${IMAGE_NAME}:${IMAGE_TAG} || true
//         //                 // docker build -t ${IMAGE_NAME}:${IMAGE_TAG} -f ${WORK_DIR}/Dockerfile ${WORK_DIR}
//         //                    docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} .
//         //             '''
//         //         }
//         //     }
//         // }
//         stage('Build Docker Image') {
//     steps {
//         dir("${WORK_DIR}") {
//             sh '''
//                 # Remove old image if exists
//                 docker rmi -f ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} || true

//                 # Build new image
//                 docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} .
//             '''
//         }
//     }
// }


//          stage('DockerHub Login') {
//             steps {
//                 withCredentials([usernamePassword(
//                     credentialsId: "${DOCKER_CREDS}",
//                     usernameVariable: 'DOCKER_USER',
//                     passwordVariable: 'DOCKER_PASS'
//                 )]) {

//                     sh """
//                     echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
//                     """
//                 }
//             }
//         }

//          stage('Push Image to DockerHub') {
//             steps {
//                 sh """
//                 docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
//                 """
//             }
//         }


//         stage('Run Docker Container') {
//     steps {
//         dir("${WORK_DIR}") {
//             sh '''
//                 docker rm -f ${CONTAINER_NAME} || true

//                 docker run -d \
//                 -p ${PORT}:${CONTAINER_PORT} \
//                 --name ${CONTAINER_NAME} \
//                 ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
//             '''
//         }
//     }
// }

//         // stage('Run Docker Container') {
//         //     steps {
//         //         dir("${WORK_DIR}") {
//         //             sh '''
//         //                 docker rm -f ${CONTAINER_NAME} || true
//         //                 // docker run -d --name ${CONTAINER_NAME} -p ${PORT}:${CONTAINER_PORT} \ ${IMAGE_NAME}:${IMAGE_TAG}
//         //                   docker run -d \
//         //         -p ${HOST_PORT}:${CONTAINER_PORT} \
//         //         --name ${CONTAINER_NAME} \
//         //         ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
//         //             '''
//         //         }
//         //     }
    

//     // post {
//     //     always {
//     //         echo "Pipeline finished! Check http://13.204.85.107:3000"
//     //     }

//         // below kubernates
//         stage('Update Kubernetes Deployment') {
//     steps {
//         sh """
//         # Set the new image for the deployment
//         kubectl set image deployment/java-deploy \
//             indie-gems-container=${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} \
//             --record

//         # Wait for rollout to finish
//         kubectl rollout status deployment/java-deploy
//         """
//     }
// }

//         ////  above kubernaties
//     }
// }









//===================================================

//
// original one is below
// pipeline {
//     agent any

//     environment {
//         WORK_DIR = "/var/lib/jenkins/workspace/Game"
//         IMAGE_NAME = "indie-gems"
//         IMAGE_TAG  = "latest"
//         CONTAINER_NAME = "indie-gems-container"
//         PORT = "50008j"   // External port for app
//     }

//     stages {
//         stage('Checkout Code') {
//             steps {
//                 dir("${WORK_DIR}") {
//                     git branch: 'main', url: 'https://github.com/SukeshKaicharla/Indie_Gems_Portal.git'
//                 }
//             }
//         }

//         stage('Build Docker Image') {
//             steps {
//                 dir("${WORK_DIR}") {
//                     sh '''
//                         docker rmi -f ${IMAGE_NAME}:${IMAGE_TAG} || true
//                         docker build -t ${IMAGE_NAME}:${IMAGE_TAG} -f ${WORK_DIR}/Dockerfile ${WORK_DIR}
//                     '''
//                 }
//             }
//         }

//         stage('Run Docker Container') {
//             steps {
//                 dir("${WORK_DIR}") {
//                     sh '''
//                         docker rm -f ${CONTAINER_NAME} || true
//                         docker run -d --name ${CONTAINER_NAME} -p ${PORT}:80 ${IMAGE_NAME}:${IMAGE_TAG}
//                     '''
//                 }
//             }
//         }
//     }

//     post {
//         always {
//             echo "Pipeline finished! Check http://13.204.85.107:3000"
//         }
//     }
// }

