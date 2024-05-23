pipeline {
  agent any

  environment {
    awsecrRegistry = '730335456215.dkr.ecr.ap-northeast-2.amazonaws.com/wordpress-ecr'
    awsecrRegistryCredential = 'credential-AWS-ECR'
    githubCredential = 'credential-github'
    gitEmail = '2071ly@gmail.com'
    gitName = 'EO7I'
  }

  options {
    disableConcurrentBuilds() // 중복 빌드 방지
  }

  triggers {
    // 여기서는 수동 트리거로 설정
  }

  stages {

    stage('Checkout Application Git Branch') {
      steps {
        checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: githubCredential, url: 'https://github.com/EO7I/Group1.git']]])
      }
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
        sh "docker build ./config -t ${awsecrRegistry}:${env.BUILD_NUMBER}"
        sh "docker build ./config -t ${awsecrRegistry}:latest"
      }
      post {
        failure {
          echo 'Docker image build failure'
        }
        success {
          echo 'Docker image build success'
        }
      }
    }

    stage('Docker Image Push') {
      steps {
        withDockerRegistry([url: "https://${awsecrRegistry}", credentialsId: awsecrRegistryCredential]) {
          sh "docker push ${awsecrRegistry}:${env.BUILD_NUMBER}"
          sh "docker push ${awsecrRegistry}:latest"
          sleep 10
        }
      }
      post {
        failure {
          echo 'Docker Image Push failure'
          sh "docker rmi ${awsecrRegistry}:${env.BUILD_NUMBER}"
          sh "docker rmi ${awsecrRegistry}:latest"
        }
        success {
          echo 'Docker Image Push success'
          sh "docker rmi ${awsecrRegistry}:${env.BUILD_NUMBER}"
          sh "docker rmi ${awsecrRegistry}:latest"
        }
      }
    }

    stage('Deploy') {
      steps {
        git credentialsId: githubCredential,
            url: 'https://github.com/EO7I/Group1.git',
            branch: 'main'

        sh "git config --global user.email ${gitEmail}"
        sh "git config --global user.name ${gitName}"
        sh "cd web && kustomize edit set image ${awsecrRegistry}:${env.BUILD_NUMBER}"
        sh "git add -A"
        sh "git status"
        sh "git commit -m 'update the image tag'"
        sh "git branch -M main"
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
