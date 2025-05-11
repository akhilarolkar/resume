pipeline {
    agent any

    parameters {
        //choice(name: 'AWS_ENV', choices: ['prod', 'preprod'], description: 'Select AWS Environment')
        string(name: 'BRANCH_NAME', choices: ['test', 'main'],defaultValue: 'test', description: 'Branch to build')
    }

    environment {
        AWS_REGION = 'eu-north-1'
        ECR_REPO = 'resume-docker'
        ECR_ACCOUNT = '320060061347'
        IMAGE_TAG = sh(script: "echo \"`date +%Y%m%d`-${BUILD_NUMBER}\"", returnStdout: true).trim()
        DOCKER_IMAGE = "${ECR_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: '${params.BRANCH_NAME}', url: 'https://github.com/akhilarolkar/resume.git'
            }
        }

        stage('Login to ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-ecr-creds']]) {
                    sh '''
                        aws configure set region $AWS_REGION
                        aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_ACCOUNT.dkr.ecr.$AWS_REGION.amazonaws.com
                    '''
                }
            }
        }

         stage('Build Docker Image') {
             steps {
                 sh 'docker build -t $DOCKER_IMAGE .'
             }
         }

         stage('Push to ECR') {
             steps {
                 sh 'docker push $DOCKER_IMAGE'
                 sh "echo \"${IMAGE_TAG}\" > image_tag.txt"
                 archiveArtifacts artifacts: 'image_tag.txt'
             }
         }
    }
}
