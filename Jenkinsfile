pipeline {
  
  agent {
    docker {
      image 'goforgold/build-container:latest'
    }
  }
  stages {
    stage('Build') {
      steps {
        sh 'npm install'
      }
    }
   stage('Create Packer AMI') {
        steps {
          withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'awsCredentials', variable: 'AWS_ACCESS_KEY_ID']]) {
            sh 'packer build -var accessKeyVariable=${AWS_ACCESS_KEY_ID} -var secretKeyVariable=${AWS_SECRET_ACCESS_KEY} packer/packer.json'   
            sh "echo this is ${env.AWS_ACCESS_KEY_ID}"
            sh "echo this is ${env.AWS_SECRET_ACCESS_KEY}"
       }
      }
    }
    stage('AWS Deployment') {
      steps {
          withCredentials([
            usernamePassword(credentialsId: 'ada90a34-30ef-47fb-8a7f-a97fe69ff93f', passwordVariable: 'AWS_SECRET', usernameVariable: 'AWS_KEY'),
            usernamePassword(credentialsId: '2facaea2-613b-4f34-9fb7-1dc2daf25c45', passwordVariable: 'REPO_PASS', usernameVariable: 'REPO_USER'),
          ]) {
            sh 'rm -rf node-app-terraform'
            sh 'git clone https://github.com/sunpyopark/node-app-terraform.git'
            sh '''
               cd node-app-terraform
               terraform init
               terraform apply -auto-approve -var access_key=${AWS_KEY} -var secret_key=${AWS_SECRET}
               git add terraform.tfstate
               git -c user.name="Shashwat Tripathi" -c user.email="shashwat2691@gmail.com" commit -m "terraform state update from Jenkins"
               git push https://${REPO_USER}:${REPO_PASS}@github.com/goforgold/node-app-terraform.git master
            '''
        }
      }
    }
  }
  environment {
    npm_config_cache = 'npm-cache'
  }
}
