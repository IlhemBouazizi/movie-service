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
                NODEPORT = '30000'
                DB_IP = '10.43.50.0'
            }
            steps 
            {
                script 
                {
                    sh '''
                    cat $KUBECONFIG > k8s_config
                    helm upgrade --kubeconfig k8s_config --install movie-microservice-$NAMESPACE  helm/movie-microservice/ --values=helm/movie-microservice/values.yaml --set nameSpace="$NAMESPACE" --set movie.service.nodePort="$NODEPORT" --set movieDB.service.clusterIP="$DB_IP"
                    '''
                }
            }
        }
        stage('Test deploiement en dev')
        {    
            environment
            {
                NODEPORT = '30000'
            }
            steps 
            {
                script 
                {
                    sh '''
                    sleep 60
                    curl "http://localhost:$NODEPORT/api/v1/movies/docs"
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
                NODEPORT = '30001'
                DB_IP = '10.43.50.1'
            }
            steps 
            {
                script 
                {
                    sh '''
                    cat $KUBECONFIG > k8s_config
                    helm upgrade --kubeconfig k8s_config --install movie-microservice-$NAMESPACE  helm/movie-microservice/ --values=helm/movie-microservice/values.yaml --set nameSpace="$NAMESPACE" --set movie.service.nodePort="$NODEPORT" --set movieDB.service.clusterIP="$DB_IP"
                    '''
                }
            }
        }
        stage('Test deploiement en qa')
        {    
            environment
            {
                NODEPORT = '30001'
            }
            steps 
            {
                script 
                {
                    sh '''
                    sleep 60
                    curl "http://localhost:$NODEPORT/api/v1/movies/docs"
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
                NODEPORT = '30002'
                DB_IP = '10.43.50.2'
            }
            steps 
            {
                script 
                {
                    sh '''
                    cat $KUBECONFIG > k8s_config
                    helm upgrade --kubeconfig k8s_config --install movie-microservice-$NAMESPACE  helm/movie-microservice/ --values=helm/movie-microservice/values.yaml --set nameSpace="$NAMESPACE" --set movie.service.nodePort="$NODEPORT" --set movieDB.service.clusterIP="$DB_IP"
                    '''
                }
            }
        }
        stage('Test deploiement en staging')
        {    
            environment
            {
                NODEPORT = '30002'
            }
            steps 
            {
                script 
                {
                    sh '''
                    sleep 60
                    curl "http://localhost:$NODEPORT/api/v1/movies/docs"
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
                NODEPORT = '30003'
                DB_IP = '10.43.50.3'
            }
            steps 
            {
                script 
                {
                    sh '''
                    cat $KUBECONFIG > k8s_config
                    helm upgrade --kubeconfig k8s_config --install movie-microservice-$NAMESPACE  helm/movie-microservice/ --values=helm/movie-microservice/values.yaml --set nameSpace="$NAMESPACE" --set movie.service.nodePort="$NODEPORT" --set movieDB.service.clusterIP="$DB_IP"
                    '''
                }
            }
        }
        stage('Test deploiement en prod')
        {    
            when 
            {
                branch 'master'
            }
            input
            {
                message "Confirmer le test en prod"
            }                        
            environment
            {
                NODEPORT = '30003'
            }
            steps 
            {
                script 
                {
                    sh '''
                    sleep 60
                    curl "http://localhost:$NODEPORT/api/v1/movies/docs"
                    '''
                }
            }            
        }        
    }
}