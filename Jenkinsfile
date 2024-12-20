pipeline {
    agent any
    environment {
        DOCKER_USER = credentials('docker-hub-credentials')
        IMAGE_NAME = 'harshalsable1/snapee-node' 
        IMAGE_TAG = "Prod-Node-${BUILD_NUMBER}" 
        BITBUCKET_USER = "harshal6-admin"
        BITBUCKET_PASS = "ATBBdB6eFbshHj3WVYuXuZnzZ3ZA841A85CD"
    }
    stages {
        
        stage('Checkout Code') {
            steps {
                script {
                    echo "Checking out code..."
                    checkout scmGit(branches: [[name: '*/Master']], extensions: [], userRemoteConfigs: [[credentialsId: 'credentials', url: 'https://harshal6-admin@bitbucket.org/harshal6/test.git']])
                }
            }
        }
        
        stage('Login to Docker Hub') {
            steps {
                script {
                    echo "Login to Docker Hub..."
                    sh 'echo $DOCKER_USER_PSW | docker login --username $DOCKER_USER_USR --password-stdin'
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker Image..."
                    sh """
                        docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                    """
                }
            }
        }
        
        stage('Cleanup Previous Images') {
            steps {
                script {
                   echo "Cleaning up previous Docker images for ${IMAGE_NAME}..."
                   sh """

                     CURRENT_SHA=\$(docker images ${IMAGE_NAME}:${IMAGE_TAG} -q)
                
                     TAG_PREFIX=\$(echo ${IMAGE_TAG} | cut -d'-' -f1,2)
                
                     docker images ${IMAGE_NAME} --format '{{.Tag}} {{.ID}}' | \\
                     grep "^\${TAG_PREFIX}" | \\
                     grep -v "${IMAGE_TAG}" | \\
                     while read tag sha; do
                       echo "Removing \${tag}..."
                     docker rmi ${IMAGE_NAME}:\${tag} || true
                     done
                
                     docker image prune -f --filter "label=stage=${IMAGE_NAME}"
                     """
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    echo "Pushing Docker Image..."
                    sh """
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }
        
        stage('Clone BitBucket Repository') {
            steps {
                script {
                    echo "Cloning target repository..."
                    withCredentials([usernamePassword(credentialsId: '7d37fdb1-b015-4ebd-b2d4-ca9685cd863f', 
                                          usernameVariable: 'BITBUCKET_USER', 
                                          passwordVariable: 'BITBUCKET_PASS')]) {
                    sh '''
                    rm -rf test
                    git clone https://${BITBUCKET_USER}:${BITBUCKET_PASS}@bitbucket.org/harshal6/test.git
                    cd test
                    '''
                    }
                }
            }
        }
        
        stage('Update Deployment Files in Prod') {
            steps {
                script {
                    echo "Updating deployment files..."
                    withCredentials([usernamePassword(credentialsId: '7d37fdb1-b015-4ebd-b2d4-ca9685cd863f', 
                                                      usernameVariable: 'BITBUCKET_USER', 
                                                      passwordVariable: 'BITBUCKET_PASS')]) {
                        sh """
                            cd test
                          
                            sed -i 's|image: .*|image: ${IMAGE_NAME}:${IMAGE_TAG}|' Node-Service/deployment.yml
                            
                            git config user.email "harshal@appscrip.co"
                            git config user.name "Harsh"
                            git add Node-Service/deployment.yml
                            git commit -m "Docker Image Update to Node Service with $IMAGE_TAG"
                            git push https://${BITBUCKET_USER}:${BITBUCKET_PASS}@bitbucket.org/harshal6/test.git Master
                        """
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline execution failed.'
        }
    }
}
