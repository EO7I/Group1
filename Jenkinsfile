pipeline {
    agent any

    // 해당 스크립트 내에서 사용할 로컬 변수들 설정
    environment {
        awsecrRegistry = '730335456215.dkr.ecr.ap-northeast-2.amazonaws.com/wordpress-ecr'
        awsecrRegistryCredential = 'credential-AWS-ECR'
        githubCredential = 'credential-github'
        gitEmail = '2071ly@gmail.com'
        gitName = 'EO7I'
    }

    stages {

        // 깃허브 계정으로 레포지토리를 클론한다.
        stage('Checkout Application Git Branch') {
            steps {
                script {
                    // Initialize the repository
                    sh 'git init'
                    sh 'git remote remove origin || true' // 기존 origin 제거 (존재하지 않아도 오류 무시)
                    sh 'git remote add origin https://github.com/EO7I/Group1.git'
                    
                    // Configure sparse checkout
                    sh 'git config core.sparseCheckout true'
                    
                    // Specify the web directory for sparse checkout
                    sh 'echo "web/" >> .git/info/sparse-checkout'
                    
                    // Pull the main branch
                    sh 'git pull origin main'
                }
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
                // 도커 이미지 빌드
                sh "docker build ./config -t ${awsecrRegistry}:${currentBuild.number}"
                sh "docker build ./config -t ${awsecrRegistry}:latest"
            }
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
                    // 10초 쉰 후에 다음 작업 이어나가도록 함
                    sleep 10
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
        stage('Update Image Tag and Commit') {
            steps {
                script {
                    // git 계정 로그인, 해당 레포지토리의 main 브랜치에서 클론
                    sh "git config --global user.email '${gitEmail}'"
                    sh "git config --global user.name '${gitName}'"
                    sh "cd web && kustomize edit set image ${awsecrRegistry}:${currentBuild.number}"
                    sh "git add -A"
                    sh "git status"
                    sh "git commit -m 'Update the Docker image tag to ${currentBuild.number}'"
                }
            }
        }

        stage('Push to Git Repository') {
            steps {
                withCredentials([usernamePassword(credentialsId: githubCredential, usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                    sh """
                    git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/EO7I/Group1.git main
                    """
                }
            }
        }
    }
}
