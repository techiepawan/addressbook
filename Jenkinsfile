pipeline{
    agent any
    options {
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '30', numToKeepStr: '2')
    }
    tools{
        maven 'Maven'
    }
    environment{
        cred = credentials('aws-cred-new')
        JAVA_HOME = '/opt/homebrew/opt/openjdk@17/libexec/openjdk.jdk/Contents/Home'
        PATH = "/usr/local/bin:$PATH"
        dockerhub_cred = credentials('docker-cred')
      // DOCKER_IMAGE = "techiepawan/project-k8s"
        DOCKER_IMAGE = "879381250162.dkr.ecr.us-east-1.amazonaws.com/privaterepo"
        DOCKER_TAG = "$BUILD_NUMBER"
    }
    stages{
        stage('Checkout stage'){
            steps{
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/techiepawan/addressbook.git']])
            }
        }
        stage('Maven Build'){
            steps{
                sh 'mvn package'
            }
        }
        stage('Docker build'){
            steps{
  //              sh 'docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .'
                sh 'docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .'
            }
        }
        stage('DockerHub push'){
            steps{
                sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 879381250162.dkr.ecr.us-east-1.amazonaws.com'
                sh 'docker push 879381250162.dkr.ecr.us-east-1.amazonaws.com/privaterepo:${BUILD_NUMBER}'
  //             sh "echo $dockerhub_cred_PSW | docker login -u $dockerhub_cred_USR --password-stdin"
                sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
                sh "docker push ${DOCKER_IMAGE}:latest"
            }
        }
        stage('Kubernetes deploy'){
            steps{
                sh 'aws eks update-kubeconfig --region us-east-1 --name devops-working'
                sh 'kubectl apply -f Application.yaml'
            }
        }
        }
    post {
        always {
            echo "Job is completed"
        }
        success {
            echo "It is a success"
        }
        failure {
            echo "Job is failed"
        }
    }
}

