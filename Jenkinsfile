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
                    helm upgrade --install cast-service charts/cast-service --namespace staging --set image.tag=$BUILD_ID --create-namespace
                    helm upgrade --install movie-service charts/movie-service --namespace staging --set image.tag=$BUILD_ID --create-namespace
                    '''
                }
            }
        }
        stage('Deploy to Prod') {
            steps {
                timeout(time: 15, unit: "MINUTES") {
                    input message: '¿Deploy in Production  Test?', ok: 'Yes'
                }
                script {
                    sh '''

                    rm -Rf .kube

                    mkdir .kube

                    cat $KUBECONFIG > .kube/config
                    helm upgrade --install cast-service charts/cast-service --namespace prod --set image.tag=$BUILD_ID --create-namespace
                    helm upgrade --install movie-service charts/movie-service --namespace prod --set image.tag=$BUILD_ID --create-namespace
                    '''
                }
            }
        }
    }
}
