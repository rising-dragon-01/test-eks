pipeline {
    agent any

    tools {
        maven 'maven3'
    }
    environment {
    //   #  DOCKERHUB_CREDENTIALS = credentials('dockerPass')
        AWS_REGION = 'ap-south-1'
        ECR_REPO = '977098995865.dkr.ecr.ap-south-1.amazonaws.com/test/first:latest'
        ECR_REPO_URL = '977098995865.dkr.ecr.ap-south-1.amazonaws.com'
        dockerImage = 'webapp01'
        // DOCKER_CREDENTIALS_ID = credentials('dockerhub')
    }
    // }

    stages {
        stage('SCM Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/rising-dragon-01/eks-deploy.git'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package'
                sh 'mv target/myweb*.war target/newapp.war'
            }
        }

        stage('SonarQube Analysis') {
            environment {
                // SonarQube server ID must match Jenkins configuration
                SONARQUBE_ENV = 'sonar'
            }
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh 'mvn sonar:sonar'
                }
            }
        }
	    stage('Quality Gate Check') {
            steps {
                sleep (60)
                timeout(time: 1, unit: 'HOURS') {
                    // Wait for the Quality Gate to be computed and check its status
               }
            }
        
        post {
 
        failure {
            echo 'sending email notification from jenkins'
    
                step([$class: 'Mailer',notifyEveryUnstableBuild: true,
                recipients: emailextrecipients([[$class: 'CulpritsRecipientProvider'],
                [$class: 'RequesterRecipientProvider']])])
                }
            }

	}
        stage('Docker Build') {
            steps {
                script {
                    // Define Docker image name and tag
                    def dockerTag = "${env.BUILD_NUMBER}"
                    // Build and tag the Docker image
                    sh "docker build -t webapp01:${dockerTag} ."  
                    // Push the Docker image to ECR
                    //sh "docker push ${ECR_REPO_URL}:${dockerTag}"
                }
            }
        }
        stage('Remove Previous Docker Image') {
            steps {
                script {
                    // Define Docker image name
                    def previousTag = env.BUILD_NUMBER.toInteger() - 1
                    // Remove the previous Docker image
                    sh "docker rmi -f ${dockerImage}:${previousTag}"
                }
            }
	}
	 stage('ECR Image Push') {
            steps {
                script {
                    def dockerTag = "${env.BUILD_NUMBER}"
                    // Define Docker image name and tag
                     sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO_URL}"
                    
                    // Tag the Docker image for ECR
                        sh "docker tag ${dockerImage}:${dockerTag} ${ECR_REPO}"
                    
                    // Push the Docker image to ECR
                    //docker push 967822984907.dkr.ecr.ap-south-1.amazonaws.com/kart:latest
                    sh "docker push ${ECR_REPO}"
                }
            }
        }
	 stage('Update Kubernetes Deployment') {
            steps {
                // Update Kubernetes deployment manifest with dynamic image tag
                script {
                    def dockerTag = "${env.BUILD_NUMBER}"
                    //sh "sed -i 's|image: my-docker-image:latest|image: ${ECR_REPO}:${dockerTag}|' main.yml"
                    sh "kubectl apply -f main.yaml"
                }
            }
        }
    }

}

