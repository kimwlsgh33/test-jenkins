pipeline {
    agent any

    triggers {
        pollSCM('*/3 * * * *')
    }

    //==============================================================
    // 환경 변수 세팅
    //==============================================================
    environment {
        AWS_ACCESS_KEY_ID = credentials('awsAccessKeyId')
        AWS_SECRET_ACCESS_KEY = credentials('awsSecretAccessKey')
        AWS_DEFAULT_REGION = 'ap-northeast-2'
        HOME = '.'
    }

    stages {
      //==============================================================
      // 빌드 전 준비
      //==============================================================
      stage('Prepare'){
        agent any

        steps {
          echo "Lets start Long Journey! ENV: ${ENV}"
          echo "Clonning Repository.."
          
          // git plugin need
          git url: 'https://github.com/kimwlsgh33/fa-frontend',
              branch: 'main',
              credentialsId: 'github_user' // jenkins credentials
        }

        post {
          // slack & email ...
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

      stage('Only for production'){
        when {
          branch 'production' // branch check
          environment name: 'APP_ENV', value: 'prod'
          anyOf {
            environment name: "DEPLOY_TO", value: 'production'
            environment name: "DEPLOY_TO", value: 'staging'
          }
        }
      }

      //==============================================================
      // aws s3 deploy
      //==============================================================
      stage('Deploy Frontend'){
        steps {
          echo "Deploying Frontend"
          // 폴더 내부로 이동후, 명령 실행
          // Lint 작업진행 필요?
          dir ('./website'){
            sh '''
            aws s3 sync ./ s3://jenkins-test
            '''
          }
        }

        post {
          success {
            echo "Successfully Cloned Repository"

            mail  to: "logosevens@gmail.com", // 보낼사람 credential 등록 필요
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

      //==============================================================
      // ECR ( AWS 권한 필요 )
      //==============================================================
      stage("Lint Backend"){
        agent {
          // docker plugin need
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

      //==============================================================
      // 
      //==============================================================
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

      //==============================================================
      // 
      //==============================================================
      stage('Build Backend'){
        agent any
        steps {
          echo "Build Backend"

          // PROD : 배표 환경에 대한, 환경 변수
          dir ("./server"){
            sh '''
            docker build . -t server --build-arg env=${PROD} // server 라는 이름으로 이미지 생성
            '''
          }
        }

        post {
          // !!!!!build가 실패하면 배포 중단!!!!!!!
          failure {
            error "This pipeline stops here..."
          }
        }

      }

      //==============================================================
      // 
      //==============================================================
      stage('Deploy Backend'){
        agent any

        steps {
          echo "Build Backend"

          dir("./server"){
            sh '''
            docker rm -f $(docker ps -aq) // docker container 끄기
            docker run -p 80:80 -d server // docker image 로 container 생성
            '''
          }
        }

        post {
          success {
            mail  to: "logosevens@gmail.com"
                  subject: "Deploy Success",
                  body: "Successfully deployed!"
          }
        }
      }
    } //<== stages
} //<== pipeline
