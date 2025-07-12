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
                docker build -t $DOCKER_ID/cast-service:$DOCKER_TAG cast-service/
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
                '''
            }
        }

        stage('Docker build - Movie service') {
            steps {
                sh '''
                docker build -t $DOCKER_ID/movie-service:$DOCKER_TAG movie-service/

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

                    helm upgrade --install cast charts/ --namespace dev --create-namespace --set image.repository=$DOCKER_ID/cast-service --set image.tag=$DOCKER_TAG --set service.nodePort=30007
                    helm upgrade --install movie charts/ --namespace dev --create-namespace --set image.repository=$DOCKER_ID/movie-service --set image.tag=$DOCKER_TAG --set service.nodePort=30008
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

                    helm upgrade --install cast charts/ --namespace qa --create-namespace --set image.repository=$DOCKER_ID/cast-service --set image.tag=$DOCKER_TAG --set service.nodePort=30017
                    helm upgrade --install movie charts/ --namespace qa --create-namespace --set image.repository=$DOCKER_ID/movie-service --set image.tag=$DOCKER_TAG --set service.nodePort=30018
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

                    helm upgrade --install cast charts/ --namespace staging --create-namespace --set image.repository=$DOCKER_ID/cast-service --set image.tag=$DOCKER_TAG --set service.nodePort=30027
                    helm upgrade --install movie charts/ --namespace staging --create-namespace --set image.repository=$DOCKER_ID/movie-service --set image.tag=$DOCKER_TAG --set service.nodePort=300028
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

                    helm upgrade --install cast charts/ --namespace prod --create-namespace --set image.repository=$DOCKER_ID/cast-service --set image.tag=$DOCKER_TAG --set service.nodePort=30037
                    helm upgrade --install movie charts/ --namespace prod --create-namespace --set image.repository=$DOCKER_ID/movie-service --set image.tag=$DOCKER_TAG --set service.nodePort=30038
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
