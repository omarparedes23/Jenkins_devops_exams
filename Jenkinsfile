pipeline {
    environment {
        DOCKER_ID = "omarelunico"
        DOCKER_PASS = credentials("DOCKER_HUB_PASS")
        KUBECONFIG = credentials("config")
    }
    agent any
    stages {
        stage('Build Docker Images') {
            steps {
                script {
                    sh '''
                    docker build -t $DOCKER_ID/cast-service:$BUILD_ID ./cast-service
                    docker build -t $DOCKER_ID/movie-service:$BUILD_ID ./movie-service
                    '''
                }
            }
        }

    stage('Verify Docker Images') {
        steps {
            script {
                sh '''
                docker images $DOCKER_ID/cast-service:$BUILD_ID
                docker images $DOCKER_ID/movie-service:$BUILD_ID
                '''
            }
        }
    }

        stage('Push to DockerHub') {
            steps {
                script {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_ID --password-stdin
                    docker push $DOCKER_ID/cast-service:$BUILD_ID
                    docker push $DOCKER_ID/movie-service:$BUILD_ID
                    '''
                }
            }
        }
        stage('Deploy to Dev') {
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    helm repo add bitnami https://charts.bitnami.com/bitnami
                    helm repo update
                    helm dependency build charts/cast-db
                    helm dependency build charts/movie-db
                    helm upgrade --install cast-db charts/cast-db --namespace dev --create-namespace
                    helm upgrade --install movie-db charts/movie-db --namespace dev --create-namespace
                    helm upgrade --install cast-service charts/cast-service --namespace dev --set image.tag=$BUILD_ID --create-namespace
                    helm upgrade --install movie-service charts/movie-service --namespace dev --set image.tag=$BUILD_ID --create-namespace
                    '''
                }
            }
        }
        stage('Deploy to QA') {
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    helm repo add bitnami https://charts.bitnami.com/bitnami
                    helm repo update
                    helm dependency build charts/cast-db
                    helm dependency build charts/movie-db
                    helm upgrade --install cast-db charts/cast-db --namespace qa --create-namespace
                    helm upgrade --install movie-db charts/movie-db --namespace qa --create-namespace
                    helm upgrade --install cast-service charts/cast-service --namespace qa --set image.tag=$BUILD_ID --create-namespace
                    helm upgrade --install movie-service charts/movie-service --namespace qa --set image.tag=$BUILD_ID --create-namespace
                    '''
                }
            }
        }
        stage('Deploy to Staging') {
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    helm repo add bitnami https://charts.bitnami.com/bitnami
                    helm repo update
                    helm dependency build charts/cast-db
                    helm dependency build charts/movie-db
                    helm upgrade --install cast-db charts/cast-db --namespace staging --create-namespace
                    helm upgrade --install movie-db charts/movie-db --namespace staging --create-namespace
                    helm upgrade --install cast-service charts/cast-service --namespace staging --set image.tag=$BUILD_ID --create-namespace
                    helm upgrade --install movie-service charts/movie-service --namespace staging --set image.tag=$BUILD_ID --create-namespace
                    '''
                }
            }
        }
        stage('Deploy to Prod') {
            when {
                branch 'origin/master'
            }
            steps {
                script {
                    echo "Deploying to production from branch: ${env.GIT_BRANCH.replace('origin/', '')}"
                }
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    helm repo add bitnami https://charts.bitnami.com/bitnami
                    helm repo update
                    helm dependency build charts/cast-db
                    helm dependency build charts/movie-db
                    helm upgrade --install cast-db charts/cast-db --namespace prod --create-namespace
                    helm upgrade --install movie-db charts/movie-db --namespace prod --create-namespace
                    helm upgrade --install cast-service charts/cast-service --namespace prod --set image.tag=$BUILD_ID --create-namespace
                    helm upgrade --install movie-service charts/movie-service --namespace prod --set image.tag=$BUILD_ID --create-namespace
                    '''
                }
            }
        }
    }
}
