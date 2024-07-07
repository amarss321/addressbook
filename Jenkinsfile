pipeline {
    agent any
    options {
     buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '30', numToKeepStr: '2')
     timeout(30)
    }
    tools {
        maven 'Maven'
    }
    environment {
        cred = credentials('aws-cred')
        IMAGE_NAME = "654654166577.dkr.ecr.us-east-1.amazonaws.com/super-project"
        IMAGE_VERSION = "$BUILD_NUMBER"
    }
    stages {
        stage('Checkout From GitHub') {
            steps {
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/amarss321/addressbook.git']])
            }
        }
        stage('SonarQube Analysis') {
            steps{
                script{
                    def mvn = tool 'Maven';
                    withSonarQubeEnv(installationName: 'sonar-server') {
                        sh "${mvn}/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=super-project"
                    }
                }
            }
        }
        stage('Build'){
            steps{
                sh 'mvn package'
            }
        }
        stage('Nexus Push'){
            steps{
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: '44.201.110.219:8081',
                    groupId: 'addressbook',
                    version: '2.0-SNAPSHOT',
                    repository: 'maven-snapshots',
                    credentialsId: 'nexus-cred',
                    artifacts: [
                        [artifactId: 'DEV',
                        classifier: '',
                        file: 'target/addressbook-2.0.war',
                        type: 'war']
                     ]
                )
            }
        }
        stage('Docker Image Build'){
            steps{
                sh 'docker build -t ${IMAGE_NAME}:v.1.${IMAGE_VERSION} .'
                sh 'docker images'
            }
        }
        stage('Push Docker Image to AWS ECR'){
            steps{
                sh '''
                    aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 654654166577.dkr.ecr.us-east-1.amazonaws.com
                    docker tag ${IMAGE_NAME}:v.1.${IMAGE_VERSION} ${IMAGE_NAME}:latest
                    docker push ${IMAGE_NAME}:latest
                '''
            }
        }
        stage('Deployin to eks'){
            steps{
                script{
                    input id: 'Deploy', message: 'All steps completed Now ready to deploy In EKS Cluster', submitter: 'admin'
                }
                sh 'kubectl version --client'
                sh 'aws eks update-kubeconfig --region us-east-1 --name eks-workshop'
                sh 'kubectl apply -f Application.yaml'
                sh 'kubectl get all -o wide'
            }
        }
    }
    post {
        always {
            echo 'Job Compeleted'
        }
        success {
            echo 'Job Successfully Compeleted'
        }
        failure {
            echo 'Job Failed'
        }
    }

}
