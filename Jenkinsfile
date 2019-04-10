pipeline {
  agent {
    docker {
      image 'diegopazosrego/node:latest'
    }

  }
  stages {
    stage('Check-PREOK') {
      steps {
        sh 'echo "OK"'
    }
    stage('Build') {
      steps {
        sh 'npm install'
      }
    }
    stage('Check') {
      steps {
        sh 'echo "OK"'
      }
    }
    stage('Create Packer AMI') {
      steps {
        withCredentials(bindings: [
                                                    usernamePassword(credentialsId: 'ada90a34-30ef-47fb-8a7f-a97fe69ff93f', passwordVariable: 'AWS_SECRET', usernameVariable: 'AWS_KEY')
                                                  ]) {
            sh 'packer build -var aws_access_key=${AWS_KEY} -var aws_secret_key=${AWS_SECRET} AMI/packer.json'
          }

        }
      }
    stage('Check-POSTOK') {
      steps {
        sh 'echo "OK"'
    }
      stage('AWS Deployment') {
        steps {
          withCredentials(bindings: [
                                                            usernamePassword(credentialsId: 'ada90a34-30ef-47fb-8a7f-a97fe69ff93f', passwordVariable: 'AWS_SECRET', usernameVariable: 'AWS_KEY'),
                                                            usernamePassword(credentialsId: '2facaea2-613b-4f34-9fb7-1dc2daf25c45', passwordVariable: 'REPO_PASS', usernameVariable: 'REPO_USER'),
                                                          ]) {
              sh 'rm -rf TFM'
              sh 'git clone https://github.com/diegopazosrego/TFM.git'
              sh '''
               cd TFM
               terraform init
               terraform apply -auto-approve -var access_key=${AWS_KEY} -var secret_key=${AWS_SECRET}
               git add terraform.tfstate
               git -c user.name="diegopazosrego" -c user.email="diegopazosrego@gmail.com" commit -m "terraform state update from Jenkins"
               git push https://${REPO_USER}:${REPO_PASS}@github.com/diegopazosrego/TFM.git master
            '''
            }

          }
        }
      }
      stage('Check-FIN') {
      steps {
        sh 'echo "OK"'
      }
      environment {
        npm_config_cache = 'npm-cache'
      }
    }
