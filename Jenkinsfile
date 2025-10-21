pipeline {
    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }    
 environment {
        SCANNER_HOME = tool 'mysonar'
        AWS_REGION = 'us-east-1' // Change to your AWS region
        ECR_REPO_APP = '585768179486.dkr.ecr.us-east-1.amazonaws.com/ravi031/myzomato'
    }
    stages {
        stage("Clean") {
            steps {
                cleanWs()
            }
        }

        stage("Code") {
            steps {
                git branch: 'main', url: 'https://github.com/RaviVarma06/Zomato-Repo.git'
            }
        }

        
       stage("CQA") {
            steps {
                withSonarQubeEnv('mysonar') {
                    sh '''mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=docker-webapp \
                        -Dsonar.projectName='docker-webapp' \
                        -Dsonar.login=sqa_7100b72f23021dcaf94bcf24be19328dba44edc8
                    '''
                }
            }
        }

        stage("Install dependencies") {
            steps {
                sh 'npm install'
            }
        }

 stage("Docker Build") {
            steps {
                script {
                    sh "docker build -t  ravi031/myzomato"
                    sh "docker tag ravi031/myzomato $ECR_REPO_APP                
                }
            }
        }

  stage("Push to ECR") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'AWS-ECR-CREDS', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh '''
                        aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                        aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                        aws configure set default.region $AWS_REGION
                        aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO_APP
                        docker push $ECR_REPO_APP
                    '''
                }
            }
        }

   stage("TrivyScan") {
            steps {
                sh 'trivy fs . > trivyfs.txt'
                sh "trivy image $ECR_REPO_APP"
            }
        }

        stage("Deploy to container"){
            steps{
	              sh 'docker run -d --name zomato -p 3000:3000 ravi031/myzomato'
	          }
      	}
    }
