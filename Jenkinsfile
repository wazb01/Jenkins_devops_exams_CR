pipeline {
    agent any

    environment {
        DOCKER_ID = "cliffrubio"
        DOCKER_TAG = "v${BUILD_ID}"
    }

    stages {
        stage('Docker build - Cast service') {
            steps {
                sh '''
                docker build -t $DOCKER_ID/cast-service:$DOCKER_TAG cast-service/
                docker tag $DOCKER_ID/cast-service:$DOCKER_TAG $DOCKER_ID/cast-service:latest
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
                docker build -t $DOCKER_ID/movie-service:$DOCKER_TAG movie-service/
                docker tag $DOCKER_ID/movie-service:$DOCKER_TAG $DOCKER_ID/movie-service:latest
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
            when {
                expression {
                    return env.BRANCH_NAME == 'dev' || env.GIT_BRANCH == 'dev' || env.GIT_BRANCH == 'origin/dev'
                }
            }
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config

                    helm upgrade --install app-dev ./helm-chart --namespace dev --create-namespace
                    '''
                }
            }
        }

        stage('Deploy to qa') {
            when {
                expression {
                    return env.BRANCH_NAME == 'qa' || env.GIT_BRANCH == 'qa' || env.GIT_BRANCH == 'origin/qa'
                }
            }
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config

                    helm upgrade --install app-qa ./helm-chart --namespace qa --create-namespace
                    '''
                }
            }
        }

        stage('Deploy to staging') {
            when {
                expression {
                    return env.BRANCH_NAME == 'staging' || env.GIT_BRANCH == 'staging' || env.GIT_BRANCH == 'origin/staging'
                }
            }
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config

                    helm upgrade --install app-staging ./helm-chart --namespace staging --create-namespace
                    '''
                }
            }
        }

        stage('Deploy to prod') {
            when {
                expression {
                    return env.BRANCH_NAME == 'master' || env.GIT_BRANCH == 'master' || env.GIT_BRANCH == 'origin/master'
                }
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

                    helm upgrade --install app-prod ./helm-chart --namespace prod --create-namespace
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
