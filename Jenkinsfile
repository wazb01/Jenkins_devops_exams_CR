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

                   helm upgrade --install cast charts/ --namespace dev --create-namespace \
                      --set image.repository=$DOCKER_ID/cast-service \
                      --set image.tag=$DOCKER_TAG \
                      --set service.nodePort=30007 \
                      --set command="{uvicorn,app.main:app,--host,0.0.0.0,--port,8000}" \
                      --set env[0].name=DATABASE_URI --set env[0].value=postgresql://cast_db_username:cast_db_password@cast_db/cast_db_dev

                    helm upgrade --install movie charts/ --namespace dev --create-namespace \
                      --set image.repository=$DOCKER_ID/movie-service \
                      --set image.tag=$DOCKER_TAG \
                      --set service.nodePort=30008 \
                      --set command="{uvicorn,app.main:app,--host,0.0.0.0,--port,8000}" \
                      --set env[0].name=DATABASE_URI --set env[0].value=postgresql://movie_db_username:movie_db_password@movie_db/movie_db_dev \
                      --set env[1].name=CAST_SERVICE_HOST_URL --set env[1].value=http://cast_service:8000/api/v1/casts/
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

                   helm upgrade --install cast charts/ --namespace qa --create-namespace \
                      --set image.repository=$DOCKER_ID/cast-service \
                      --set image.tag=$DOCKER_TAG \
                      --set service.nodePort=30007 \
                      --set command="{uvicorn,app.main:app,--host,0.0.0.0,--port,8000}" \
                      --set env[0].name=DATABASE_URI --set env[0].value=postgresql://cast_db_username:cast_db_password@cast_db/cast_db_dev

                    helm upgrade --install movie charts/ --namespace qa --create-namespace \
                      --set image.repository=$DOCKER_ID/movie-service \
                      --set image.tag=$DOCKER_TAG \
                      --set service.nodePort=30008 \
                      --set command="{uvicorn,app.main:app,--host,0.0.0.0,--port,8000}" \
                      --set env[0].name=DATABASE_URI --set env[0].value=postgresql://movie_db_username:movie_db_password@movie_db/movie_db_dev \
                      --set env[1].name=CAST_SERVICE_HOST_URL --set env[1].value=http://cast_service:8000/api/v1/casts/
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

                   helm upgrade --install cast charts/ --namespace staging --create-namespace \
                      --set image.repository=$DOCKER_ID/cast-service \
                      --set image.tag=$DOCKER_TAG \
                      --set service.nodePort=30007 \
                      --set command="{uvicorn,app.main:app,--host,0.0.0.0,--port,8000}" \
                      --set env[0].name=DATABASE_URI --set env[0].value=postgresql://cast_db_username:cast_db_password@cast_db/cast_db_dev

                    helm upgrade --install movie charts/ --namespace staging --create-namespace \
                      --set image.repository=$DOCKER_ID/movie-service \
                      --set image.tag=$DOCKER_TAG \
                      --set service.nodePort=30008 \
                      --set command="{uvicorn,app.main:app,--host,0.0.0.0,--port,8000}" \
                      --set env[0].name=DATABASE_URI --set env[0].value=postgresql://movie_db_username:movie_db_password@movie_db/movie_db_dev \
                      --set env[1].name=CAST_SERVICE_HOST_URL --set env[1].value=http://cast_service:8000/api/v1/casts/
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

                   helm upgrade --install cast charts/ --namespace prod --create-namespace \
                      --set image.repository=$DOCKER_ID/cast-service \
                      --set image.tag=$DOCKER_TAG \
                      --set service.nodePort=30007 \
                      --set command="{uvicorn,app.main:app,--host,0.0.0.0,--port,8000}" \
                      --set env[0].name=DATABASE_URI --set env[0].value=postgresql://cast_db_username:cast_db_password@cast_db/cast_db_dev

                    helm upgrade --install movie charts/ --namespace prod --create-namespace \
                      --set image.repository=$DOCKER_ID/movie-service \
                      --set image.tag=$DOCKER_TAG \
                      --set service.nodePort=30008 \
                      --set command="{uvicorn,app.main:app,--host,0.0.0.0,--port,8000}" \
                      --set env[0].name=DATABASE_URI --set env[0].value=postgresql://movie_db_username:movie_db_password@movie_db/movie_db_dev \
                      --set env[1].name=CAST_SERVICE_HOST_URL --set env[1].value=http://cast_service:8000/api/v1/casts/
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
