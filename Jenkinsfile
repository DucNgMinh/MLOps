pipeline {
    agent any

    options{
        // Max number of build logs to keep and days to keep
        buildDiscarder(logRotator(numToKeepStr: '5', daysToKeepStr: '5'))
        // Enable timestamp at each job in the pipeline
        timestamps()
    }

    environment{
        registry = 'ducngminh/bank-campaign-api'
        registryCredential = 'dockerhub'
    }

    stages {
        stage('Test') {
            agent {
                docker {
                    image 'python:3.9'
                }
            }
            steps {
                echo 'Testing model correctness..'
                sh 'pip install -r requirements.txt'
                sh 'pip install pytest'
                sh 'pytest'
            }
        }
        stage('Build') {
            steps {
                script {
                    echo 'Building image for deployment..'
                    dockerImage = docker.build registry + ":$BUILD_NUMBER"
                    echo 'Pushing image to dockerhub..'
                    docker.withRegistry( '', registryCredential ) {
                        dockerImage.push()
                        dockerImage.push('latest')
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying models..'
                sh '''
                    # Check if container is running and stop it
                    if [ $(docker ps -q -f name=bank-campaign-api) ]; then
                        echo "Stopping existing bank-campaign-api container"
                        docker stop bank-campaign-api
                        docker rm bank-campaign-api
                    fi
                    
                    # Pull latest image and start container
                    docker pull ducngminh/bank-campaign-api:latest
                    docker run -d --name bank-campaign-api -p 8000:8000 ducngminh/bank-campaign-api:latest
                '''
            }
        }
    }
}
