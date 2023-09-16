pipeline 
{
    agent any // Jenkins will be able to select all available agents
    stages 
    {
        stage('Pre-Build')
        { //we pass the built image to our docker hub account
            environment
            {
                DOCKER_PASS = credentials("DOCKER_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
                DOCKER_ID = 'ilhemb'
                BRANCH_NAME = "${GIT_BRANCH.split("/")[1]}"
            }
            steps 
            {
                script 
                {
                    sh '''
                    echo "Pulling $BRANCH_NAME branch"

                    git branch
                    
                    docker login -u $DOCKER_ID -p $DOCKER_PASS
                    '''
                }
            }
        }
        stage('Build')
        { // docker build image stage
            steps {
                script 
                {
                    sh '''
                    echo ""
                    echo "-----  Build movie service"
                    docker image build webapp/movie-service/ -t movie-service:latest
                    docker tag movie-service ilhemb/movie-service:latest
                    docker image push ilhemb/movie-service:latest
                    '''
                }
            }
        }
        stage('Deploiement en dev')
        {
            environment
            {
                KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
                NAMESPACE = 'dev'
                CHARTNAME = 'movie-microservice-dev'                
                CLUSTERIP_MOVIE = '10.43.50.0'
            }
            steps 
            {
                script 
                {
                    sh '''
                    cat $KUBECONFIG > k8s_config
                    helm upgrade --kubeconfig k8s_config --install $CHARTNAME  helm/movie-microservice/ --values=helm/movie-microservice/values.yaml --set nameSpace="$NAMESPACE"  --set movie.service.clusterIP="$CLUSTERIP_MOVIE" 
                    '''
                }
            }
        }
        stage('Deploiement en qa')
        {
            environment
            {
                KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
                NAMESPACE = 'qa'
                CHARTNAME = 'movie-microservice-qa'                
                CLUSTERIP_MOVIE = '10.43.51.0'
            }
            steps 
            {
                script 
                {
                    sh '''
                    cat $KUBECONFIG > k8s_config
                    helm upgrade --kubeconfig k8s_config --install $CHARTNAME  helm/movie-microservice/ --values=helm/movie-microservice/values.yaml --set nameSpace="$NAMESPACE"  --set movie.service.clusterIP="$CLUSTERIP_MOVIE" 
                    '''
                }
            }
        }
        stage('Deploiement en staging')
        {
            environment
            {
                KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
                NAMESPACE = 'staging'
                CHARTNAME = 'movie-microservice-staging'                
                CLUSTERIP_MOVIE = '10.43.52.0'
            }
            steps 
            {
                script 
                {
                    sh '''
                    cat $KUBECONFIG > k8s_config
                    helm upgrade --kubeconfig k8s_config --install $CHARTNAME  helm/movie-microservice/ --values=helm/movie-microservice/values.yaml --set nameSpace="$NAMESPACE"  --set movie.service.clusterIP="$CLUSTERIP_MOVIE" 
                    '''
                }
            }
        }
        stage('Deploiement en prod')
        {
            when 
            {
                branch 'master'
            }            
            input
            {
                message "Confirmer le deployment en prod"
            }
            environment
            {
                KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
                NAMESPACE = 'prod'
                CHARTNAME = 'movie-microservice-prod'                
                CLUSTERIP_MOVIE = '10.43.53.0'
            }
            steps 
            {
                script 
                {
                    sh '''
                    cat $KUBECONFIG > k8s_config
                    helm upgrade --kubeconfig k8s_config --install $CHARTNAME  helm/movie-microservice/ --values=helm/movie-microservice/values.yaml --set nameSpace="$NAMESPACE"  --set movie.service.clusterIP="$CLUSTERIP_MOVIE" 
                    '''
                }
            }
        }
    }
}