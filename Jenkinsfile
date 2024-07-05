pipeline {
    agent any
    options {
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '20', numToKeepStr: '2')
    }
    tools {
        maven 'Maven'
    }
    environment {
        cred = credentials('aws-cred')
        DOCKER_IMAGE = "590184028154.dkr.ecr.us-west-2.amazonaws.com/addressbook"
        DOCKER_TAG = "$BUILD_NUMBER"
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/amarss321/addressbook.git']])
            }
        }
        stage('SonarQube Analysis') {
            steps{
                script{
                    def mvn = tool 'Maven';
                    withSonarQubeEnv(installationName: 'sonar') {
                        sh "${mvn}/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=addressbook"
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
                    nexusUrl: '52.42.195.62:80',
                    groupId: 'addressbook',
                    version: '2.0-SNAPSHOT',
                    repository: 'maven-snapshots',
                    credentialsId: 'nexus-cred',
                    artifacts: [
                        [artifactId: 'TEST',
                        classifier: '',
                        file: 'target/addressbook-2.0.war',
                        type: 'war']
                     ]
                )
            }
        }
        stage('Docker image Build'){
            steps{
                sh 'docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .'
            }
        }
        stage('Docker image Push to ECR'){
            steps{
                sh '''
                    aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin 590184028154.dkr.ecr.us-west-2.amazonaws.com
                    docker tag  ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                    docker push ${DOCKER_IMAGE}:latest
                '''
            }
        }
    }
    post {
        always {
            echo 'Job compeleted'
    }
        success {
            echo 'Job successfully created image is pushed to ECR'
    }
        failure {
            echo 'job Failed'
    }
}

}
