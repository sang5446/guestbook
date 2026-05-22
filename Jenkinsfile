import java.text.SimpleDateFormat

def TODAY = (new SimpleDateFormat("yyyyMMddHHmmss")).format(new Date())

pipeline {
    agent any
    environment {
        strDockerTag = "${TODAY}_${BUILD_ID}"
        strDockerImage ="sang5446/cicd_guestbook:${strDockerTag}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url:'https://github.com/sang5446/guestbook.git'

                dir('/root/sub-workspace/guestbook-config'){
                    git branch: 'master', url:'https://github.com/sang5446/guestbook-config.git'
                }
            }
        }

        stage('Build') {
            steps {
                sh './mvnw clean package'
            }
        }

        stage('Docker Image Build') {
            steps {
                script {
                    oDockImage = docker.build(strDockerImage, "--build-arg VERSION=${strDockerTag} -f Dockerfile .")
                }
            }
        }

        stage('Docker Image Push') {
            steps {
                script {
                    docker.withRegistry('', 'DockerHub_Credential') {
                        oDockImage.push()
                    }
                }
            }
        }

        stage('Config-Repo PUSH') {
            environment {
                GITHUB_ACCESS_TOKEN = credentials('github-access-token')
            }
            steps {
                dir('/root/sub-workspace/guestbook-config'){
                    sh '''
                        sed -i "s/cicd_guestbook:.*/cicd_guestbook:${strDockerTag}/g" guestbook/guestbook_deploy.yaml
                        git add guestbook/guestbook_deploy.yaml
                        git commit -m "[UPDATE] guestbook image tag - ${strDockerImage} (by jenkins)"
                        git push "https://sang5446:${GITHUB_ACCESS_TOKEN}@github.com/sang5446/guestbook-config.git"
                    '''
                }
            }
        }

        stage('ArgoCD Sync') {
            environment {
                ARGOCD_API_TOKEN = credentials('argocd-api-token')
            }
            steps {
                sh '''
                    TOKEN="${ARGOCD_API_TOKEN}"
                    PAYLOAD='{"prune": true}'
                    curl -v -k -XPOST \
                        -H "Authorization: Bearer ${TOKEN}" \
                        https://172.31.1.200/api/v1/applications/guestbook/sync
                '''
            }
        }
    }
    post { 
        success { 
            slackSend(tokenCredentialId: 'slack-token'
                , channel: '#교육'
                , color: 'good'
                , message: "상소영-  ${JOB_NAME} (${BUILD_NUMBER}) 빌드 성공! 너무 유익한 수업이었습니다. 감사합니다. Details: (<${BUILD_URL} | here >)")
        }
        failure { 
            slackSend(tokenCredentialId: 'slack-token'
                , channel: '#교육'
                , color: 'danger'
                , message: "상소영 - ${JOB_NAME} (${BUILD_NUMBER}) 빌드가 실패하였습니다. Details: (<${BUILD_URL} | here >)")
    }
  }
}

