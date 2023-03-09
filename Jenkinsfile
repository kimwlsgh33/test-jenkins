pipeline {
    agent any

    triggers {
        pollSCM('*/3 * * * *')
    }

    environment {
        AWS_ACCESS_KEY_ID = credentials('awsAccessKeyId')
        AWS_SECRET_ACCESS_KEY = credentials('awsSecretAccessKey')
        AWS_DEFAULT_REGION = 'ap-northeast-2'
        HOME = '.'
    }

    stages {
      stage('Prepare'){
        agent any

        steps {
          echo "Clonning Repository.."
          git url: 'https://github.com/kimwlsgh33/test-jenkins.git',
              branch: 'main',
              credentialsId: 'github_user'
        }

        post {
          success {
            echo "Successfully Cloned Repository"
          }

          always {
            echo "i tried..."
          }

          cleanup {
            echo "after all other post condition"
          }
        }
      }

      stage('Deploy Frontend'){
        steps {
          echo "Deploying Frontend"
          dir ('./website'){
            sh '''
            aws s3 sync ./ s3://jenkins-test-03
            '''
          }
        }

        post {
          success {
            echo "Successfully Cloned Repository"

            mail  to: "logosevens@gmail.com", 
                  subject: "Deploy Frontend Success",
                  body: "Successfully deployd frontend!"
          }

          failure {
            echo "I failed :("

            mail  to: "logosevens@gmail.com",
                  subject: "Failed Pipelinee",
                  body: "something is wrong with deploy frontend"
          }
        }

      }

      stage("Lint Backend"){
        agent {
          docker {
            image "node:latest"
          }
        }

        steps {
          dir ('./server'){
            sh '''
              npm install&&
              npm run lint
            '''
          }
        }
      }

      stage("Test Backend"){
        agent {
          docker {
            image "node:latest"
          }
        }
        steps {
          echo "Test Backend"

          dir ("./server"){
            sh '''
            npm install
            npm run test
            '''
          }
        }
      }

      stage('Build Backend'){
        agent any
        steps {
          echo "Build Backend"

          dir ("./server"){
            sh '''
            docker build . -t server --build-arg env=${PROD}
            '''
          }
        }

        post {
          failure {
            error "This pipeline stops here..."
          }
        }

      }

      stage('Deploy Backend'){
        agent any

        steps {
          echo "Build Backend"

          dir("./server"){
            sh '''
            docker rm -f $(docker ps -aq)
            docker run -p 80:80 -d server
            '''
          }
        }

        post {
          success {
            mail  to: "logosevens@gmail.com",
                  subject: "Deploy Success",
                  body: "Successfully deployed!"
          }
        }
      }
    } //<== stages
} //<== pipeline
