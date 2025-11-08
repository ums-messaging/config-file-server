pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-northeast-2'
        TARGET_INSTANCE_ID = 'i-01865e024d7b04019'
        REGISTRY = "registry.ums.local:5000"
        APP_NAME = "config-server"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        GIT_BRANCH = "${env.BRANCH_NAME ?: 'main'}"
    }

    stages {
        stage('Checkout Gateway Server') {
            steps {
                 script {
                    def branch = env.BRANCH_NAME ?: 'main'
                    sshagent(['git']) {
                         sh """
                            if [ ! -d .git ]; then
                                git init
                                git remote add origin git@github.com:ums-messaging/config-file-server.git
                                git pull origin $GIT_BRANCH
                            else
                                git fetch origin $GIT_BRANCH
                                git checkout $GIT_BRANCH
                                git pull origin $GIT_BRANCH
                            fi
                         """
                    }
                 }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    aws s3 cp /var/jenkins_home/workspace/config-file-server/ s3://ums-config-file-bucket/ums-config-file/ --recursive
                    curl -X POST http://localhost:8888/actuator/busrefresh
                '''
            }
        }
    }

    post {
        failure {
            echo "❌ 빌드 또는 배포 실패!"
        }
        success {
            echo "✅ 전체 파이프라인 완료!"
        }
    }
}
