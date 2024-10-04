pipeline {
    environment {
        DOCKER_ID = "alex0489" 
        DOCKER_IMAGE_CAST = "cast-service"
        DOCKER_IMAGE_MOVIE = "movie-service"
        DOCKER_TAG = "v.${BUILD_ID}.0" // Tagging with the build number for unique identification
    }
    agent any 
    stages {  // Begin stages block
        stage('Docker Build') { // Docker build images stage
            steps {
                script {
                    sh '''
                    docker rm -f jenkins || true
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG ./cast-service
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG ./movie-service
                    sleep 6
                    '''
                }
            }
        }

        stage('Docker Push') { // Push built images to DockerHub
            environment {
                DOCKER_PASS = credentials("DOCKER_PASS") // Docker password saved in Jenkins credentials
            }
            steps {
                script {
                    sh '''
                    docker login -u $DOCKER_ID -p $DOCKER_PASS
                    docker push $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG
                    docker push $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG
                    '''
                }
            }
        }

        stage('Deployment on dev') {
            // when {
            //     branch 'dev' // Deploy only when changes are pushed to the 'dev' branch
            // }
            environment {
                KUBECONFIG = credentials("config") // Retrieve kubeconfig from Jenkins credentials
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config

                    # Deploy cast-service
                    cp /home/ubuntu/jenkins_EVAL/Jenkins_devops_exams/cast-service/castapp-chart/values-dev.yaml values-cast.yaml
                    sed -i "s+tag:.*+tag: ${DOCKER_TAG}+g" values-cast.yaml
                    helm upgrade --install cast-service /home/ubuntu/jenkins_EVAL/Jenkins_devops_exams/cast-service/castapp-chart --values=values-cast.yaml --namespace dev --create-namespace || true

                    # Check if the deployment was successful
                    if ! kubectl rollout status deployment/cast-service-cast-service -n dev && ! kubectl rollout status deployment/cast-service-db -n dev; then
                        echo "Deployment failed. Rolling back..."
                        helm rollback cast-service -n dev
                        exit 1
                    fi

                    # Deploy movie-service
                    cp /home/ubuntu/jenkins_EVAL/Jenkins_devops_exams/movie-service/movieapp-chart/values-dev.yaml values-movie.yaml
                    sed -i "s+tag:.*+tag: ${DOCKER_TAG}+g" values-movie.yaml
                    helm upgrade --install movie-service /home/ubuntu/jenkins_EVAL/Jenkins_devops_exams/movie-service/movieapp-chart --values=values-movie.yaml --namespace dev --create-namespace || true

                    # Check if the deployment was successful
                    if ! kubectl rollout status deployment/movie-service-movie-service -n dev && ! kubectl rollout status deployment/movie-service-db -n dev; then
                        echo "Deployment failed. Rolling back..."
                        helm rollback movie-service -n dev
                        exit 1
                    fi
                    '''
                }
            }
        }

        stage('Deployment on staging') {
            // when {
            //     branch 'staging' // Deploy only when changes are pushed to the 'staging' branch
            // }
            environment {
                KUBECONFIG = credentials("config") // Retrieve kubeconfig from Jenkins credentials
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config

                    # Deploy cast-service
                    cp /home/ubuntu/jenkins_EVAL/Jenkins_devops_exams/cast-service/castapp-chart/values-staging.yaml values-cast.yaml
                    sed -i "s+tag:.*+tag: ${DOCKER_TAG}+g" values-cast.yaml
                    helm upgrade --install cast-service /home/ubuntu/jenkins_EVAL/Jenkins_devops_exams/cast-service/castapp-chart --values=values-cast.yaml --namespace staging --create-namespace || true

                    # Check if the deployment was successful
                    if ! kubectl rollout status deployment/cast-service-cast-service -n staging && ! kubectl rollout status deployment/cast-service-db -n staging; then
                        echo "Deployment failed. Rolling back..."
                        helm rollback cast-service -n staging
                        exit 1
                    fi

                    # Deploy movie-service
                    cp /home/ubuntu/jenkins_EVAL/Jenkins_devops_exams/movie-service/movieapp-chart/values-staging.yaml values-movie.yaml
                    sed -i "s+tag:.*+tag: ${DOCKER_TAG}+g" values-movie.yaml
                    helm upgrade --install movie-service /home/ubuntu/jenkins_EVAL/Jenkins_devops_exams/movie-service/movieapp-chart --values=values-movie.yaml --namespace staging --create-namespace || true

                    # Check if the deployment was successful
                    if ! kubectl rollout status deployment/movie-service-movie-service -n staging && ! kubectl rollout status deployment/movie-service-db -n staging; then
                        echo "Deployment failed. Rolling back..."
                        helm rollback movie-service -n staging
                        exit 1
                    fi
                    '''
                }
            }
        }

        stage('Deployment on prod') {
            // when {
            //     branch 'main' // Deploy only when changes are pushed to the 'main' branch
            // }
            environment {
                KUBECONFIG = credentials("config") // Retrieve kubeconfig from Jenkins credentials
            }
            steps {
                // Manual approval before deploying to production
                timeout(time: 15, unit: "MINUTES") {
                    input message: 'Do you want to deploy in production?', ok: 'Yes'
                }

                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config

                    # Deploy cast-service
                    cp /home/ubuntu/jenkins_EVAL/Jenkins_devops_exams/cast-service/castapp-chart/values-prod.yaml values-cast.yaml
                    sed -i "s+tag:.*+tag: ${DOCKER_TAG}+g" values-cast.yaml
                    helm upgrade --install cast-service /home/ubuntu/jenkins_EVAL/Jenkins_devops_exams/cast-service/castapp-chart --values=values-cast.yaml --namespace prod --create-namespace || true

                    # Check if the deployment was successful
                    if ! kubectl rollout status deployment/cast-service-cast-service -n prod && ! kubectl rollout status deployment/cast-service-db -n prod; then
                        echo "Deployment failed. Rolling back..."
                        helm rollback cast-service -n prod
                        exit 1
                    fi

                    # Deploy movie-service
                    cp /home/ubuntu/jenkins_EVAL/Jenkins_devops_exams/movie-service/movieapp-chart/values-prod.yaml values-movie.yaml
                    sed -i "s+tag:.*+tag: ${DOCKER_TAG}+g" values-movie.yaml
                    helm upgrade --install movie-service /home/ubuntu/jenkins_EVAL/Jenkins_devops_exams/movie-service/movieapp-chart --values=values-movie.yaml --namespace prod --create-namespace || true

                    # Check if the deployment was successful
                    if ! kubectl rollout status deployment/movie-service-movie-service -n prod && ! kubectl rollout status deployment/movie-service-db -n prod; then
                        echo "Deployment failed. Rolling back..."
                        helm rollback movie-service -n prod
                        exit 1
                    fi
                    '''
                }
            }
        }
    } // End stages block
} // End pipeline block
