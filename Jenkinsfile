pipeline {
    agent any

    stages {
        stage('Clone git repo') {
            steps {
                sh 'echo "STAGE 0: Cloning app code from SCM ..."'
                git 'https://github.com/mudathirlawal/cicd-with-jenkins-docker-and-aws-eks.git'
            }    
        }        
        stage('Lint all app code') {
            steps {
                sh 'echo "STAGE 1: Checking app code for syntax error ..."'
                sh 'tidy -q -e *.html'
            }
        }   
        stage( 'Build docker image for app' ) {
            steps {
                sh 'echo "STAGE 2: Building and tagging docker image ..."'
                sh 'docker build -t web-app:v1.0 .'
                sh 'docker image ls'                  
            }
        } 
        stage( 'Push image to dockerhub repo' ) {
            steps {
                withDockerRegistry([url: "", credentialsId: "dockerhub"]) {
                    sh 'echo "STAGE 3: Uploading image to dockerhub repository ..."'
                    sh 'docker login'
                    sh 'docker tag web-app:v1.0 nigercode/web-app:v1.0'
                    sh 'docker push nigercode/web-app:v1.0'          
                }
            }
        }                                   
        stage( 'Deploy image to AWS EKS' ) {
            steps {
                withAWS( region:'us-west-2', credentials:'capstone' ) {
                    sh 'echo "STAGE 4: Deploying image to AWS EKS cluster ..."'
                    sh 'aws eks --region us-west-2 update-kubeconfig --name capstone --role-arn arn:aws:iam::428819381342:role/capstoneEksClusterRole'                   
                    sh 'kubectl config use-context arn:aws:cloudformation:us-west-2:428819381342:stack/capstone-ctrl-plane/5a748780-b2c1-11ea-b7cb-02ca0801967a'
                    sh 'kubectl set image deployments/web-app web-app=nigercode/web-app:v1.0'
                    sh 'kubectl apply -f deployment/deployment.yml'
                    sh 'kubectl get nodes'
                    sh 'kubectl get deployment'
                    sh 'kubectl get pod -o wide'
                    sh 'kubectl get services/web-app'
                    sh 'echo "Congratulations! Deployment successful."'
                    sh 'kubectl decribe deployment/nigercode/web-app'
                }
            }
        }               
    }
}
