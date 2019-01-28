pipeline {
  agent none
  stages {
    stage('Checkout') {
      agent any
      steps {
        echo 'Checking out master'
        git(url: 'https://github.com/bponath/ecs-demo-php-simple-app', branch: 'master')
      }
    }
    stage('BuildImage') {
      agent any
      steps {
        echo 'Building image'
        sh '''
            printenv
            export AWS_DEFAULT_REGION=us-east-1
            $(aws ecr get-login --no-include-email --region us-east-1)
            docker build -t docker-ci-test:$GIT_COMMIT .
            docker tag docker-ci-test:$GIT_COMMIT 574233067113.dkr.ecr.us-east-1.amazonaws.com/ecsbl-repos-1bi6yqfr6rvyf:$GIT_COMMIT
            docker push 574233067113.dkr.ecr.us-east-1.amazonaws.com/ecsbl-repos-1bi6yqfr6rvyf:$GIT_COMMIT
        '''
        milestone 1
      }
    }
    stage('DeployImageToStage') {
      agent any
      steps {
        echo 'Deploying new image'
        sh '''
            STACKNAME='ECSBlueGreen-DeploymentPipeline-1FVOAQ764ONXS-ecs-cluster'
            aws s3 cp s3://brianlab-ecs-bluegreen/scripts/deployer_jenkins.py deployer_jenkins.py
            aws s3 cp --recursive s3://brianlab-ecs-bluegreen/templates/ templates/
            python3 deployer_jenkins.py $STACKNAME
        '''
        milestone 2
      }
    }
    stage('Approval') {
      agent none
      input {
        message 'Approve deployment?'
      }
      steps {
        milestone 3
        echo 'Approved'
      }
    }
    stage('FlipStageToProduction') {
      agent any
      steps {
        echo 'Flipping stage to prod'
        sh '''
            ALB='ecs-demo-php-simple-app'
            aws s3 cp s3://brianlab-ecs-bluegreen/scripts/blue_green_flip_jenkins.py blue_green_flip_jenkins.py
            python3 blue_green_flip_jenkins.py $ALB
        '''
        milestone 4
      }
    }
  }
}