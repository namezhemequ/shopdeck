# Jenkins CI/CD Pipeline for mall-admin

## 前置准备：配置 GitHub SSH 密钥

Jenkins 拉取私有仓库需要 SSH 密钥：

1. 在 Jenkins 容器内生成密钥：
   `docker exec -it mall-jenkins ssh-keygen -t ed25519 -C "jenkins@shopdeck"`

2. 查看公钥：
   `docker exec mall-jenkins cat /var/jenkins_home/.ssh/id_ed25519.pub`

3. 添加到 GitHub：https://github.com/namezhemequ/shopdeck/settings/keys → Deploy keys → Add

## Pipeline Job 配置

### 1. 新建 Item → Pipeline → 名称: mall-admin-cicd

### 2. Pipeline script（复制以下内容）:

```groovy
pipeline {
    agent any
    
    environment {
        IMAGE_NAME = 'mall-admin'
        IMAGE_TAG  = "${BUILD_NUMBER}"
        PREV_TAG   = "${BUILD_NUMBER - 1}"
        DEPLOY_DIR = '/home/jenkins-deploy'
    }
    
    stages {
        stage('Checkout') {
            steps {
                sh '''
                    if [ -d .git ]; then
                        git fetch origin && git checkout -f origin/master
                    else
                        git clone -b master git@github.com:namezhemequ/shopdeck.git .
                    fi
                '''
            }
        }
        
        stage('Docker Build') {
            steps {
                sh '''
                    docker build \
                        -t ${IMAGE_NAME}:${IMAGE_TAG} \
                        -f mall-admin/Dockerfile .
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
                '''
            }
        }
        
        stage('Deploy') {
            steps {
                sh '''
                    cd ${DEPLOY_DIR}
                    docker-compose up -d --no-deps admin
                '''
            }
        }
        
        stage('Health Check') {
            steps {
                script {
                    def maxRetries = 12
                    def retry = 0
                    def healthy = false
                    
                    while (retry < maxRetries && !healthy) {
                        sleep(5)
                        retry++
                        try {
                            def result = sh(
                                script: 'curl -s http://admin:8080/actuator/health',
                                returnStdout: true
                            ).trim()
                            if (result.contains('UP')) {
                                healthy = true
                                echo "Health check passed after ${retry * 5}s"
                            }
                        } catch (Exception e) {
                            echo "Attempt ${retry}: not ready yet..."
                        }
                    }
                    
                    if (!healthy) {
                        error("Health check failed! Rolling back...")
                    }
                }
            }
        }
    }
    
    post {
        failure {
            script {
                echo "Pipeline failed! Rolling back to previous image..."
                sh '''
                    cd ${DEPLOY_DIR}
                    docker tag ${IMAGE_NAME}:${PREV_TAG} ${IMAGE_NAME}:latest || true
                    docker-compose up -d --no-deps admin
                '''
            }
        }
        success {
            echo "Deploy successful! Image: ${IMAGE_NAME}:${IMAGE_TAG}"
        }
    }
}
```

### 触发方式

- **手动触发**: Build Now
- **定时轮询**: Build Triggers → Poll SCM → `H/5 * * * *`（每 5 分钟检查 git 更新）
- **Webhook**（需公网可达）: GitHub repo → Settings → Webhooks → Payload URL: `http://<服务器IP>:8090/github-webhook/`
