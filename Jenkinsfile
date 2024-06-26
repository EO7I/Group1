pipeline {
  agent any

  // 로컬 변수 설정
  // 레포지토리가 없으면 생성됨
  // Credential에는 젠킨스에서 설정한 ID를 사용
  environment {
    awsecrRegistry = '730335456215.dkr.ecr.ap-northeast-2.amazonaws.com/wordpress-ecr'
    awsecrRegistryCredential = 'credential-AWS-ECR'
    githubCredential = 'credential-github'
    gitEmail = '2071ly@gmail.com'
    gitName = 'EO7I'
  }
  
  
  stages {

    // 깃허브 계정 레포지토리 클론
    stage('Checkout Application Git Branch') {
      steps {
        checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: githubCredential, url: 'https://github.com/EO7I/Group1.git']]])
      }
      // 실패시 failure 성공시 success 실행
      post {
        failure {
          echo 'Repository clone failure'
        }
        success {
          echo 'Repository clone success'
        }
      }
    }

    stage('Docker Image Build') {
      steps {
        // 도커 이미지 빌드
        sh "docker build ./config -t ${awsecrRegistry}:${currentBuild.number}"
        sh "docker build ./config -t ${awsecrRegistry}:latest"
      }
      // 성공, 실패 시 슬랙에 알람오도록 설정
      post {
        failure {
          echo 'Docker image build failure'
          //slackSend (color: '#FF0000', message: "FAILED: Docker Image Build '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
        success {
          echo 'Docker image build success'
          //slackSend (color: '#0AC9FF', message: "SUCCESS: Docker Image Build '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
      }
    }

    stage('Docker Image Push') {
      steps {
        // 젠킨스에 등록한 계정으로 ECR 에 이미지 푸시
        withDockerRegistry([url: "https://${awsecrRegistry}", credentialsId: "ecr:ap-northeast-2:${awsecrRegistryCredential}"]) {
          sh "docker push ${awsecrRegistry}:${currentBuild.number}"
          sh "docker push ${awsecrRegistry}:latest"
          // 11초 쉰 후에 다음 작업 이어나가도록 함
          sleep 11
        }
      }
      post {
        failure {
          echo 'Docker Image Push failure'
          sh "docker rmi ${awsecrRegistry}:${currentBuild.number}"
          sh "docker rmi ${awsecrRegistry}:latest"
          //slackSend (color: '#FF0000', message: "FAILED: Docker Image Push '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
        success {
          echo 'Docker Image Push success'
          sh "docker rmi ${awsecrRegistry}:${currentBuild.number}"
          sh "docker rmi ${awsecrRegistry}:latest"
          //slackSend (color: '#0AC9FF', message: "SUCCESS: Docker Image Push '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
      }
    }

    // updated docker image 태그를 git push
    stage('Deploy') {
      steps {
        // git 계정 로그인, 해당 레포지토리의 main 브랜치에서 클론
        git credentialsId: githubCredential,
            url: 'https://github.com/EO7I/Group1.git',
            branch: 'main'

        // 이미지 태그 변경 후 메인 브랜치에 푸시
        sh "git config --global user.email ${gitEmail}"
        sh "git config --global user.name ${gitName}"
        sh "cd web && kustomize edit set image ${awsecrRegistry}:${currentBuild.number}"
        sh "git add -A"
        sh "git status"
        sh "git commit -m 'update the image tag'"
        
        // update-web-folder 브랜치로 체크아웃
        sh "git checkout update-web-folder"
                
        // 만약 local에 해당 브랜치가 없을 때 원격 브랜치에서 체크아웃
        sh "git checkout -B update-web-folder origin/update-web-folder"
              }
    }

    stage('Push to Git Repository') {
      steps {
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: githubCredential, usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD']]) {
                 sh "git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/EO7I/Group1.git"

        }
      }
    }
  }
}
