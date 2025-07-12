pipeline {
    environment {
        DOCKER_ID = "cliffrubio"
        DOCKER_IMAGE = "fastapiapp-jenkins"
        DOCKER_TAG = "v${BUILD_ID}"
        BRANCH_NAME = "${env.GIT_BRANCH}".replace('origin/', '')
    }

    agent any

    stages {
        stage('Docker Build') {
            steps {
                script {
                    sh '''
                        docker rm -f jenkins
                        docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG .
                    '''
                }
            }
        }

        stage('Docker Push') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }
            steps {
                script {
                    sh '''
                        docker login -u $DOCKER_ID -p $DOCKER_PASS
                        docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                    '''
                }
            }
        }

        stage('Deploy to dev') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config

                    cp charts/values.yaml values.yaml
                    sed -i "s|repository:.*|repository: $DOCKER_ID/$DOCKER_IMAGE|" values.yaml
                    sed -i "s|tag:.*|tag: $DOCKER_TAG|" values.yaml

                    helm upgrade --install myapp charts/ --values values.yaml --namespace dev --create-namespace
                    '''
                }
            }
        }

        stage('Deploy to qa') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config

                    cp charts/values.yaml values.yaml
                    sed -i "s|repository:.*|repository: $DOCKER_ID/$DOCKER_IMAGE|" values.yaml
                    sed -i "s|tag:.*|tag: $DOCKER_TAG|" values.yaml

                    helm upgrade --install myapp charts/ --values values.yaml --namespace qa --create-namespace
                    '''
                }
            }
        }

        stage('Deploy to staging') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config

                    cp charts/values.yaml values.yaml
                    sed -i "s|repository:.*|repository: $DOCKER_ID/$DOCKER_IMAGE|" values.yaml
                    sed -i "s|tag:.*|tag: $DOCKER_TAG|" values.yaml

                    helm upgrade --install myapp charts/ --values values.yaml --namespace staging --create-namespace
                    '''
                }
            }
        }

        stage('Deploy to prod') {
            when {
                branch 'master'
            }
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Do you want to deploy in production ?', ok: 'Yes'
                }

                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config

                    cp charts/values.yaml values.yaml
                    sed -i "s|repository:.*|repository: $DOCKER_ID/$DOCKER_IMAGE|" values.yaml
                    sed -i "s|tag:.*|tag: $DOCKER_TAG|" values.yaml

                    helm upgrade --install myapp charts/ --values values.yaml --namespace prod --create-namespace
                    '''
                }
            }
        }
    }

    post {
        failure {
            mail to: 'cliffrubio.dev@gmail.com',
                subject: "${env.JOB_NAME} - Build # ${env.BUILD_ID} has failed",
                body: "For more info on the pipeline failure, check out the console output at ${env.BUILD_URL}"
        }
    }
}
