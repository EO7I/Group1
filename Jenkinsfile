pipeline {
  agent any

  environment {
    awsecrRegistry = '730335456215.dkr.ecr.ap-northeast-2.amazonaws.com/wordpress-ecr'
    awsecrRegistryCredential = 'credential-AWS-ECR'
    githubCredential = 'credential-github'
    gitEmail = '2071ly@gmail.com'
    gitName = 'EO7I'
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
        sh "docker build ./config -t ${awsecrRegistry}:${currentBuild.number}"
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
        withDockerRegistry([url: "https://${awsecrRegistry}", credentialsId: "ecr:ap-northeast-2:${awsecrRegistryCredential}"]) {
          sh "docker push ${awsecrRegistry}:${currentBuild.number}"
          sh "docker push ${awsecrRegistry}:latest"
          sleep 10
        }
      }
      post {
        failure {
          echo 'Docker Image Push failure'
          sh "docker rmi ${awsecrRegistry}:${currentBuild.number}"
          sh "docker rmi ${awsecrRegistry}:latest"
        }
        success {
          echo 'Docker Image Push success'
          sh "docker rmi ${awsecrRegistry}:${currentBuild.number}"
          sh "docker rmi ${awsecrRegistry}:latest"
        }
      }
    }

    stage('Update Image Tag and Commit') {
      steps {
        script {
          sh "git config --global user.email '${gitEmail}'"
          sh "git config --global user.name '${gitName}'"
          sh "cd web && kustomize edit set image ${awsecrRegistry}:${currentBuild.number}"
          sh "cd web && git add ."
          sh "git commit -m 'Update the Docker image tag to ${currentBuild.number}'"
        }
      }
    }

    stage('Push to Git Repository') {
      steps {
        withCredentials([usernamePassword(credentialsId: githubCredential, usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
          script {
           sh """
              git pull https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/EO7I/Group1.git main
              git subtree push --prefix=web https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/EO7I/Group1.git main
            """
          }
        }
      }
    }
  }
}
