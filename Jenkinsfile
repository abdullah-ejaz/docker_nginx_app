def app = 'Unknown'
pipeline{
    agent any
    environment {
    AWS_DEFAULT_REGION = 'us-east-2'
    AWS_ACCOUNT_ID="489994096722"
    IMAGE_REPO_NAME="abdullah_jenkins_ecr"
    IMAGE_TAG="${env.BUILD_ID}"
    REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"   
    Public_Subnet_2 = "subnet-09c93874"
    Public_Subnet_1 = "subnet-eaa81381"
    // service_name = "appinservice"
    parameters {
        string(name: 'ENVIRONMENT', defaultValue: 'DEV', description: 'Where should I deploy?')
        choice(name: 'service_name', choices: ['appinservice', 'pythonapp'], description: 'Pick Service to Deploy')
    }    
    TASK_FAMILY="task"
    }    
    stages{  
        stage('Logging into AWS ECR') {
            steps {
                script {
                    sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
                }
            }
        }         
        stage("Build image"){
            steps{
                script {
                    app = docker.build"${IMAGE_REPO_NAME}:${IMAGE_TAG}"
                }
            }   
        }
        stage("Test image"){
            when {
                expression { 
                    return params.ENVIRONMENT == 'DEV'
                }            
            steps{
                script {
                    app.inside {            
                        sh 'echo "Tests passed"'        
                    } 
                }
            }   
        }   
        stage("Push image"){
            steps{
                script {
                    sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:$IMAGE_TAG"
                    sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
                }
            }
        }
        stage("register Task defintion"){
            steps{
                script {
                    sh "sed -i 's/%BUILD_ID%/${BUILD_ID}/g' task.json"
                    //  sed -i 's/database_name_here/wordpress/g' /var/www/html/wp-config.php
                    sh "aws ecs register-task-definition --cli-input-json file://task.json"
                }
            }
        }     
        stage("Update service"){
            steps{
                script {
                    sh "aws ecs update-service --cluster abdullah-jenkins-fargate --service ${params.service_name} --task-definition task --desired-count 1"
                }
            }
        }                     
    }
}  
