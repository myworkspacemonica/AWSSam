pipeline {
  agent any
      environment {
        BRANCH_NAME_WITHOUT_TYPE = "${GIT_BRANCH.split('/').size() > 1? GIT_BRANCH.split('/')[1..-1].join('/'): GIT_BRANCH}" 
        AWS_REGION = 'ca-central-1'
        DEV_STACK_NAME = 'aws-sam-dev-deploy'
        DEV_S3_BUCKET = 'awssammonica'
        DEV_ENV= "env-dev-vars.txt"
      }

  stages {
    stage('Install Pipeline Dependencies') {
      steps {
        sh 'python3 -m venv venv && venv/bin/pip install aws-sam-cli && venv/bin/pip install pytest'
        stash includes: '**/venv/**/*', name: 'venv'
      }
    }
    stage('Build') {
      steps {
        unstash 'venv'
        sh 'venv/bin/pytest -v'
        sh 'venv/bin/sam build'
        stash includes: '**/.aws-sam/**/*', name: 'aws-sam'
      }
    }
    stage('Deploy to main') {
      
      when {
        expression {BRANCH_NAME_WITHOUT_TYPE == 'main'}
      }
      steps {
        withCredentials([
          [
            $class: 'AwsSamJenkinsCredentials',
            credentialsId: "aws_sam_jenkins_deploy_dev",
            accessKeyVariable: 'AWS_SAM_ACCESS_KEY_ID',
            secretKeyVariable: 'AWS_SAM_SECRETE_KEY_ID'
          ]
        ]) {
          unstash 'venv'
          unstash 'aws-sam'
          sh 'venv/bin/sam deploy --stack-name $DEV_STACK_NAME -t template.yml --s3-bucket $DEV_S3_BUCKET --region $AWS_REGION --capabilities CAPABILITY_NAMED_IAM --parameter-overrides $(cat ${DEV_ENV})'
          
          }
        }
      }
  } 
}       
   
