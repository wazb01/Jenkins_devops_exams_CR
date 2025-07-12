pipeline {
    environment {
        DOCKER_ID = "cliffrubio"
        DOCKER_TAG = "v${BUILD_ID}"
    }

    agent any

    stages {
        stage('Docker build - Cast service') {
            steps {
                sh '''
                docker build -t $DOCKER_ID/cast-service:$DOCKER_TAG -t $DOCKER_ID/cast-service:latest cast-service/
                '''
            }
        }

        stage('Docker push - Cast service') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }
            steps {
                sh '''
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push $DOCKER_ID/cast-service:$DOCKER_TAG
                docker push $DOCKER_ID/cast-service:latest
                '''
            }
        }

        stage('Docker build - Movie service') {
            steps {
                sh '''
                docker build -t $DOCKER_ID/movie-service:$DOCKER_TAG -t $DOCKER_ID/movie-service:latest movie-service/
                '''
            }
        }

        stage('Docker push - Movie service') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }
            steps {
                sh '''
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push $DOCKER_ID/movie-service:$DOCKER_TAG
                docker push $DOCKER_ID/movie-service:latest
                '''
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

                    helm upgrade --install cast charts/ --namespace dev --create-namespace --set image.repository=$DOCKER_ID/cast-service --set image.tag=$DOCKER_TAG
                    helm upgrade --install movie charts/ --namespace dev --create-namespace --set image.repository=$DOCKER_ID/movie-service --set image.tag=$DOCKER_TAG
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

                    helm upgrade --install cast charts/ --namespace dev --create-namespace --set image.repository=$DOCKER_ID/cast-service --set image.tag=$DOCKER_TAG
                    helm upgrade --install movie charts/ --namespace dev --create-namespace --set image.repository=$DOCKER_ID/movie-service --set image.tag=$DOCKER_TAG
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

                    helm upgrade --install cast charts/ --namespace dev --create-namespace --set image.repository=$DOCKER_ID/cast-service --set image.tag=$DOCKER_TAG
                    helm upgrade --install movie charts/ --namespace dev --create-namespace --set image.repository=$DOCKER_ID/movie-service --set image.tag=$DOCKER_TAG
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

                    helm upgrade --install cast charts/ --namespace dev --create-namespace --set image.repository=$DOCKER_ID/cast-service --set image.tag=$DOCKER_TAG
                    helm upgrade --install movie charts/ --namespace dev --create-namespace --set image.repository=$DOCKER_ID/movie-service --set image.tag=$DOCKER_TAG
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
