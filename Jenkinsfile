pipeline {
  agent any

  parameters {
    booleanParam(name : 'BUILD_DOCKER_IMAGE', defaultValue : true, description : 'BUILD_DOCKER_IMAGE')
    booleanParam(name : 'RUN_TEST', defaultValue : true, description : 'RUN_TEST')
    booleanParam(name : 'PUSH_DOCKER_IMAGE', defaultValue : true, description : 'PUSH_DOCKER_IMAGE')
    booleanParam(name : 'DEPLOY_WORKLOAD', defaultValue : true, description : 'DEPLOY_WORKLOAD')

    string(name : 'AWS_ACCOUNT_ID', defaultValue : '546703828218', description : 'AWS_ACCOUNT_ID')
    string(name : 'DOCKER_IMAGE_NAME', defaultValue : 'castle_ecr', description : 'DOCKER_IMAGE_NAME')
    string(name : 'DOCKER_TAG', defaultValue : '1', description : 'DOCKER_TAG')

  }


  environment {
    REGION = "ap-northeast-2"
    ECR_REPOSITORY = "${params.AWS_ACCOUNT_ID}.dkr.ecr.ap-northeast-2.amazonaws.com"
    ECR_DOCKER_IMAGE = "${ECR_REPOSITORY}/${params.DOCKER_IMAGE_NAME}"
    ECR_DOCKER_TAG = "${params.DOCKER_TAG}"
  }

  stages {
    stage('============ Build Docker Image ============') {
        when {
            expression { return params.BUILD_DOCKER_IMAGE }
        }
        steps {
            dir("${env.WORKSPACE}") {
                sh 'docker build -t ${ECR_DOCKER_IMAGE}:${ECR_DOCKER_TAG} .'
            }
        }
        post {
            always {
                echo "Docker build success!"
            }
        }
    }

    stage('============ Push Docker Image ============') {
        when { expression { return params.PUSH_DOCKER_IMAGE } }
        steps {
            echo "Push Docker Image to ECR"
            sh'''
                aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${ECR_REPOSITORY}
                docker push ${ECR_DOCKER_IMAGE}:${ECR_DOCKER_TAG}

            '''
              }
         }
    stage('============ Deploy workload ============') {
        when { expression { return params.DEPLOY_WORKLOAD } }
        steps {
                    sh "kubectl apply -f deployment.yaml"
                }
            }
     }
}
