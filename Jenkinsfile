pipeline {
    environment {
        DOCKER_ID = "omarelunico"  // Tu ID de DockerHub
        DOCKER_PASS = credentials("DOCKER_HUB_PASS")  // Credencial para DockerHub
        KUBECONFIG = credentials("config")  // Credencial para el archivo kubeconfig
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
                branch 'master'
            }
            steps {
                script {
                    echo "Deploying to production from branch: ${env.GIT_BRANCH.replace('origin/', '')}"
                }
                timeout(time: 15, unit: "MINUTES") {
                    input message: '¿Deploy in Production?', ok: 'Yes'
                }
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
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
