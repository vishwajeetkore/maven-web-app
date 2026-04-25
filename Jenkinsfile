pipeline {
    agent any 
    
    environment {
        AWS_REGION = 'us-east-1'
        ACCOUNT_ID = '531262218012'
        ECR_REPO = 'mavenwebapp'
        IMAGE_TAG = 'latest'
        ECR_URI = "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}"
    }
    
    stages {
        stage ('Git Clone'){
            steps {
                git credentialsId: 'vishwajeetGitHub', 
                url: 'https://github.com/vishwajeetkore/maven-web-app.git'
            }
        }

        stage ('Build'){
            steps {
                sh 'mvn clean package'
            }
        }

        stage ('Docker Build'){
            steps {
                sh "docker build -t ${ECR_REPO}:${IMAGE_TAG} ."
            }
        }

        stage ('Login to ECR'){
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                credentialsId: 'aws-cred']]){
                    
                    sh '''
                    aws ecr get-login-password --region $AWS_REGION | \
                    docker login --username AWS --password-stdin $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                    '''
                }
            }
        }

        stage ('Tag & Push'){
            steps {
                sh """
                docker tag ${ECR_REPO}:${IMAGE_TAG} ${ECR_URI}:${IMAGE_TAG}
                docker push ${ECR_URI}:${IMAGE_TAG}
                """
            }
        }
        
        stage('Deploy'){
            steps {
                sh """
                aws eks update-kubeconfig --region us-east-1 --name loanreapay

                kubectl get nodes
            
                kubectl apply -f k8s-deploy.yml
                
                # force update because using latest
                kubectl rollout restart deployment/mavenwebappdeployment
                
                kubectl rollout status deployment/mavenwebappdeployment
                """
            }
        }
    }
}
