import groovy.json.JsonOutput

pipeline {
    agent any

    environment {
        // GIT
        TAG_ID = "${BUILD_ID}"
        NS = "devops"
        GIT_TOKEN = credentials('git_token_xxx')
        GIT_CRED = "git_xxx"
        REPO_NAME = "fe-core-ticketing"
        GIT_URL = "https://github.com/xxx/${REPO_NAME}.git"
        GIT_CONFIG_URL = "https://github.com/xxx/core-deployment-config.git"
        BRANCH_CONFIG = "devops"
        BRANCH_REPO = "development"
        // Apps
        // DELETE = "false"        //////////////////////W A R N I N G///////////////////////
        // PATH = "go-api-v1"
        APP_NAME = "hello-world"
        IMAGE_NAME = "mahesaj/${APP_NAME}:${TAG_ID}"
        DOCKER_REGISTRY = "https://index.docker.io/v1/"
        DOCKER_CREDENTIALS_ID = "dockerhub-devops"
        KUBECONFIG = credentials('kubernetes')
        KUBECONFIG_PATH = "${WORKSPACE}/cd_config"
        CON_PORT = "8666"
        SVC_PORT = "8666"
    }

    stages {

        stage('Clean Workspace') {
            steps {
                echo "Cleaning workspace..."
                cleanWs()
                sh "tree -lah"
            }
        }

        stage('SCM Checkout') {
            parallel {
                stage('App Repo') {
                    steps {
                        echo "Cloning application repository..."
                        checkout scmGit(
                            branches: [[name: "*/${BRANCH_REPO}"]], 
                            extensions: [
                                // [$class: 'SparseCheckoutPaths', sparseCheckoutPaths: [
                                //     [$class: 'SparseCheckoutPath', path: "${PATH}"]
                                // ]]
                            ], 
                            userRemoteConfigs: [[credentialsId: "${GIT_CRED}", url: "${GIT_URL}" ]]
                        )
                        sh "pwd"
                        sh "ls -lah"
                        sh "tree || true"
                        sh "pwd"
                    }
                }

                stage('Config Repo') {
                    steps {
                        echo "Cloning configuration repository..."
                        checkout scmGit(
                            branches: [[name: "*/${BRANCH_CONFIG}"]],
                            extensions: [
                                [$class: 'SparseCheckoutPaths', sparseCheckoutPaths: [
                                    [$class: 'SparseCheckoutPath', path: 'cd_config'],
                                    [$class: 'SparseCheckoutPath', path: "${REPO_NAME}"]
                                ]],
                                [$class: 'RelativeTargetDirectory', relativeTargetDir: 'secrets']
                            ],
                            userRemoteConfigs: [[credentialsId: "${GIT_CRED}", url: "${GIT_CONFIG_URL}"]]
                        )
                        sh """
                            pwd
                            ls -lah
                            tree -lah
                            ls -lah
                            mv secrets/cd_config ${WORKSPACE}/
                            mv secrets/${REPO_NAME}/k8s ${WORKSPACE}/
                            mv secrets/${REPO_NAME}/configs ${WORKSPACE}/
                            rm -rf secrets secrets@tmp
                            ls -lah
                            tree -lah
                            ls -lah
                        """
                    }
                }
            }
        }

        stage('Set Kubernetes Context') {
            steps {
                echo "Setting Kubernetes context to 'dev'..."
                sh "tree -lah"
                sh "pwd"
                sh "ls -lah"
                sh ('kubectl --kubeconfig ${KUBECONFIG} config use-context dev')
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building and pushing Docker image..."
                script {
                    sh "ls "
                    dockerImage = docker.build("${IMAGE_NAME}", ".")
                }
            }
        }

        stage('Test Run Container') {
            steps {
                echo "Running container for testing..."
                script {
                    sh """
                        docker rm -f ${APP_NAME} || true
                        docker run -dp ${CON_PORT}:${CON_PORT} --name ${APP_NAME} ${IMAGE_NAME}
                        sleep 5 
                        docker logs ${APP_NAME}
                        curl -v localhost:${CON_PORT} || true
                        docker exec -it ${APP_NAME} sh || ls -lah || pwd
                    """
                    // sh " sleep 10"
                }
            }
        }

        // stage('Cleanup Test Container') {
        //     steps {
        //         echo "Cleaning up test container and image..."
        //         script {
        //             sh "docker rm -f ${APP_NAME} || true"
        //             // sh "docker rmi ${IMAGE_NAME} || true"
        //         }
        //     }
        // }

        // stage('Push Docker Image') {
        //     steps {
        //         echo "Pushing Docker image..."
        //         script {
        //             docker.withRegistry("${DOCKER_REGISTRY}", "${DOCKER_CREDENTIALS_ID}") {
        //                 dockerImage.push()
        //             sh "docker rmi ${IMAGE_NAME} || true"
        //             }
        //         }
        //     }
        // }

        stage('Update Manifests TAG, PORT, Etc') {
            steps {
                script {
                    // sh "tree"
                    // sh "sed -i 's/__TAG__/${TAG_ID}/g' k8s/k8s/deployment.yaml"
                    // sh "sed -i 's/__CONTAINERPORT__/${CON_PORT}/g' k8s/k8s/deployment.yaml"
                    // sh "cat k8s/k8s/deployment.yaml"
                    sh "tree -lah"
                    sh """
                        export __TAG__=${TAG_ID}
                        export __APPNAME__=${APP_NAME}
                        export __CONTAINERPORT__=${CON_PORT}
                        export __SVCPORT__=${SVC_PORT}
                        envsubst < k8s/k8s/deployment.yaml.tpl > k8s/k8s/deployment.yaml
                        envsubst < k8s/k8s/service.yaml.tpl > k8s/k8s/service.yaml
                    """
                    sh "cat k8s/k8s/deployment.yaml k8s/k8s/service.yaml"
                }
            }
        }

        // stage('Deploy to Kubernetes') {
        //     steps {
        //         script {
        //             echo "Deploying to Kubernetes..."
        //             sh "kubectl apply -f k8s/k8s/deployment.yaml --kubeconfig ${KUBECONFIG_PATH}"
        //             sh "kubectl apply -f k8s/k8s/service.yaml --kubeconfig ${KUBECONFIG_PATH}"
        //             sh "sleep 15"
        //         }
        //     }
        // }

        // stage('Check Deployment & Delete RS') {
        //     steps {
        //         script {
        //             echo "Checking deployment & delete rs status..."
        //             def rsList = sh(script: "kubectl -n ${NS} delete rs \$(kubectl -n ${NS} get rs | awk '{if (\$2 + \$3 + \$4 == 0) print \$1}' | grep -v 'NAME')", returnStdout: true).trim()
        //             echo "Deleted RS: ${rsList}"
        //             if (rsList) {
        //                 sh "kubectl -n ${NS} delete rs ${rsList}"
        //             } else {
        //                 echo "No RS to delete"
        //             }
        //             sh "kubectl get all -n ${NS}"
        //         }
        //     }
        // }

        // stage('Check Deployment & Delete RS') {
        //     steps {
        //         script {
        //             echo "Checking deployment & delete rs status..."
        //             // Get the list of ReplicaSets and store it in a variable
        //             def rsList = sh(script: "kubectl -n ${NS} get rs -o jsonpath='{.items[*].metadata.name}'", returnStdout: true).trim()
                    
        //             // Check if the rsList is empty
        //             if (rsList) {
        //                 // If there are ReplicaSets, proceed to delete them
        //                 echo "Deleting ReplicaSets: ${rsList}"
        //                 sh "kubectl -n ${NS} delete rs ${rsList}"
        //             } else {
        //                 // If no ReplicaSets are found, log a message and skip deletion
        //                 echo "No ReplicaSets found to delete."
        //             }
                    
        //             // Get the current status of all resources in the namespace
        //             sh "kubectl get all -n ${NS}"
        //         }
        //     }
        // }
        

        // stage('Notify Slack') {
        //     steps {
        //         script {
        //             def payload = JsonOutput.toJson([
        //                 channel: env.notification_channel,
        //                 attachments: [
        //                     [
        //                         color: "#36a64f",
        //                         author_name: "Jenkins",
        //                         title: "Deployment Status",
        //                         text: "Deployment of ${APP_NAME} is successful!",
        //                         fields: [
        //                             [
        //                                 title: "Project",
        //                                 value: "${APP_NAME}",
        //                                 short: true
        //                             ],
        //                             [
        //                                 title: "Build Number",
        //                                 value: "${BUILD_NUMBER}",
        //                                 short: true
        //                             ],
        //                             [
        //                                 title: "Build ID",
        //                                 value: "${BUILD_ID}",
        //                                 short: true
        //                             ],
        //                             [
        //                                 title: "Build URL",
        //                                 value: "${BUILD_URL}",
        //                                 short: false
        //                             ]
        //                         ]
        //                     ]
        //                 ]
        //             ])
        //             slackSend color: "#36a64f", message: "Deployment of ${APP_NAME} is successful!", payload: payload, tokenCredentialId: env.SLACK_TOKEN
        //         }
        //     }
        // }

        // stage('Notify Telegram') {
        //     steps {
        //         script {
        //             def payload = JsonOutput.toJson([
        //                 chatId: env.TELEGRAM_CHAT_ID,
        //                 text: "Deployment of ${APP_NAME} is successful!"
        //             ])
        //             telegramSend message: "Deployment of ${APP_NAME} is successful!", payload: payload, tokenCredentialId: env.TELEGRAM_TOKEN
        //         }
        //     }
        // }

        // stage('Update Service YAML') {
        //     steps {
        //         script {
        //             sh "tree -lah"
        //             sh "sed -i 's/__APPNAME__/${APP_NAME}/g' k8s/k8s/service.yaml"
        //             sh "sed -i 's/__SVCPORT__/${SVC_PORT}/g' k8s/k8s/service.yaml"
        //             sh "sed -i 's/__CONTAINERPORT__/${CON_PORT}/g' k8s/k8s/service.yaml"
        //             sh "cat k8s/k8s/service.yaml"
        //         }
        //     }
        // }

        // stage('Helm Upgrade or Install') {
        //     steps {
        //         script {
        //             echo "Deploying ${APP_NAME} via Helm..."
        //             sh """
        //                 helm upgrade --install ${APP_NAME} ./k8s/helm-chart \\
        //                     --namespace ${NS} \\
        //                     --set image.repository=mahesaj/${APP_NAME} \\
        //                     --set image.tag=${TAG_ID} \\
        //                     --set service.port=${SVC_PORT} \\
        //                     --kubeconfig ${KUBECONFIG_PATH}
        //             """
        //         }
        //     }
        // }

        // stage('Helm Upgrade or Install') {
        //     steps {
        //         script {
        //             echo "Deploying to Kubernetes using Helm..."
        //             sh "ls -lah k8s/"
        //             sh """
        //                 helm upgrade --install ${APP_NAME} ./k8s/helm-chart \\
        //                     --namespace default \\
        //                     --set image.repository=mahesaj/${APP_NAME} \\
        //                     --set image.tag=${TAG_ID} \\
        //                     --set service.port=${PORT} \\
        //                     --kubeconfig ${KUBECONFIG_PATH}
        //             """
        //             sh "helm list --namespace default --kubeconfig ${KUBECONFIG_PATH}"
        //         }
        //     }
        // }
        // stage('Delete Old Deployments') {
        //     steps {
        //         script {
        //             if (DELETE == "true") {
        //                 echo "Deleting old deployments..."
        //                 sh "kubectl delete deployment --all -n ${NS} --kubeconfig ${KUBECONFIG_PATH}"
        //                 sh "kubectl delete service --all -n ${NS} --kubeconfig ${KUBECONFIG_PATH}"
        //                 sh "kubectl delete rs --all -n ${NS} --kubeconfig ${KUBECONFIG_PATH}"
        //             } else {
        //                 echo "Skipping deletion of old deployments..."
        //             }
        //         }
        //     }
        // }

    }
}
