pipeline {
    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }

    environment {
        SCANNER_HOME = tool 'mysonar'
    }

    stages {
        stage("Clean") {
            steps {
                cleanWs()
            }
        }

        stage("Code") {
            steps {
                git "https://github.com/RaviVarma06/dockerwebapp.git"
            }
        }

        stage("CQA") {
            steps {
                withSonarQubeEnv('mysonar') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=docker-webapp \
                        -Dsonar.projectKey=docker-webapp
						-Dsonar.java.binaries=target/classe
                       '''
                }
            }
        }
        stage("Build") {
            steps {
                sh 'mvn clean package'
				sh 'cp -r target Docker-app'
            }
        }
        stage("OWASP") {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage("Docker Build") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'DOCKER-HUB') {
                        sh "docker build -t ravi031/ourproject:app Docker-app"
                        sh "docker build -t ravi031/ourproject:db Docker-db"
                    }
                }
            }
        }

        stage("TrivyScan") {
            steps {
                sh 'trivy fs . > trivyfs.txt'
                sh "trivy image ravi031/ourproject:app"
                sh "trivy image ravi031/ourproject:db"
            }
        }

        stage("DHpush") {
            steps {
                withDockerRegistry(credentialsId: 'DOCKER-HUB', url: 'https://index.docker.io/v1/') {
                    sh 'docker push ravi031/ourproject:app'
                    sh 'docker push ravi031/ourproject:db'
                }
            }
        }

        stage("Deploy to container"){
            steps{
	             sh 'docker-compose up -d' 
	        }
     	}
    }
}
