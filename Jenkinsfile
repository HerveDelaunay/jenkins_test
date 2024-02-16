pipeline {
environment { // Declaration of environment variables
DOCKER_ID = "hdelaunay" // replace this with your docker-id
CAST_IMAGE = "cast-app"
MOVIE_IMAGE = "movie-app"
DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
}
agent any // Jenkins will be able to select all available agents
stages {
        stage(' Docker Build'){ // docker build image stage
            steps {
                script {
                sh '''
                 docker rm -f jenkins
                 docker build -t $DOCKER_ID/$CAST_IMAGE:$DOCKER_TAG ./cast-service
                 docker build -t $DOCKER_ID/$MOVIE_IMAGE:$DOCKER_TAG ./movie-service
                sleep 6
                '''
                }
            }
        }
        stage('Docker run'){ // run container from our builded image
                steps {
                    script {
                    sh '''
                    docker run -d -p 80:8000 --rm --name jenkins-cast $DOCKER_ID/$CAST_IMAGE:$DOCKER_TAG
                    docker run -d -p 81:8000 --rm --name jenkins-movie $DOCKER_ID/$MOVIE_IMAGE:$DOCKER_TAG
                    sleep 10
                    '''
                    }
                }
            }

        stage('Test Acceptance'){ // we launch the curl command to validate that the container responds to the request
            steps {
                    script {
                    sh '''
                    curl localhost:80
		    curl localhost:80
                    '''
                    }
            }

        }
        stage('Docker Push'){ //we pass the built image to our docker hub account
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
            }

            steps {

                script {
                sh '''
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push $DOCKER_ID/$MOVIE_IMAGE:$DOCKER_TAG
                docker push $DOCKER_ID/$CAST_IMAGE:$DOCKER_TAG
                '''
                }
            }

        }

stage('Deploiement en dev'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                 sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yaml > tmp.yaml
		 cat tmp.yaml > values.yaml
		 rm tmp.yaml
                helm upgrade --install app . --values=values.yaml --namespace dev
                '''
                }
            }

        }
stage('Deploiement en staging'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                 sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yaml > tmp.yaml
		 cat tmp.yaml > values.yaml
		 rm tmp.yaml
                helm upgrade --install app . --values=values.yaml --namespace staging
                '''
                }
            }

        }
  stage('Deploiement en qa'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {

                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                 sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yaml > tmp.yaml
		 cat tmp.yaml > values.yaml
		 rm tmp.yaml
                helm upgrade --install app . --values=values.yaml --namespace qa
                '''
                }
            }

        }
  stage('Deploiement en prod'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
            // Create an Approval Button with a timeout of 15minutes.
            // this require a manuel validation in order to deploy on production environment
                    timeout(time: 15, unit: "MINUTES") {
                        input message: 'Do you want to deploy in production ?', ok: 'Yes'
                    }

                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                 sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yaml > tmp.yaml
		 cat tmp.yaml > values.yaml
		 rm tmp.yaml
                helm upgrade --install app . --values=values.yaml --namespace prod
                '''
                }
            }

        }

}
}
