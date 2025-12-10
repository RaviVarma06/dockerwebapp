pipeline {
    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node16'
        maven 'mymaven'
    }

    environment {
        SCANNER_HOME = tool 'mysonar'
        AWS_REGION = 'ap-south-1'
        ECR_REPO_APP = '904923506382.dkr.ecr.ap-south-1.amazonaws.com/ravi031/myzomato'
        ECR_REPO_DB = '904923506382.dkr.ecr.ap-south-1.amazonaws.com/ravi031/myzomato'
    }

    stages {
        stage("Clean") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout Code") {
            steps {
                git "https://github.com/RaviVarma06/dockerwebapp.git"
            }
        }

        stage("Code Quality Analysis") {
            steps {
                withSonarQubeEnv('mysonar') {
                    sh '''
                        mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=docker-webapp \
                        -Dsonar.projectName='docker-webapp' \
                        -Dsonar.login=sqa_c57711f61c5845601a8678702ba858b460cb3b1b
                    '''
                }
            }
        }

        stage("Build WAR") {
            steps {
                sh 'mvn clean package'
            }
        }

        stage("Docker Build") {
            steps {
                script {
                    sh "DOCKER_BUILDKIT=1 docker build -t ravi031/ourproject:app -f Docker-app/Dockerfile ."
                    sh "DOCKER_BUILDKIT=1 docker build -t ravi031/ourproject:db -f Docker-db/Dockerfile ."
                    sh "docker tag ravi031/ourproject:app $ECR_REPO_APP"
                    sh "docker tag ravi031/ourproject:db $ECR_REPO_DB"
                }
            }
        }

        stage("Push to ECR") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'AWS-ECR-CRED', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh '''
                        aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                        aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                        aws configure set default.region $AWS_REGION
                        aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin 904923506382.dkr.ecr.ap-south-1.amazonaws.com
                        docker push $ECR_REPO_APP
                        docker push $ECR_REPO_DB
                    '''
                }
            }
        }

        stage("Trivy Scan") {
            steps {
                sh 'trivy fs . > trivyfs.txt'
                sh "trivy image $ECR_REPO_APP"
                sh "trivy image $ECR_REPO_DB"
            }
        }

        stage("Deploy to Container") {
            steps {
                 sh 'docker-compose up -d'
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution completed.'
        }
        failure {
            echo 'Pipeline failed. Please check the logs.'
        }
    }
}
