pipeline {
    agent any

    environment {
        AWS_REGION     = 'ap-southeast-1'
        REPO_PHP       = '147925596024.dkr.ecr.ap-southeast-1.amazonaws.com/quiz-php-fpm'
        REPO_NGINX     = '147925596024.dkr.ecr.ap-southeast-1.amazonaws.com/quiz-nginx'
        APP_DIR        = '.'
        IMAGE_TAG      = "${BUILD_ID}"
    }

    stages {
        // === 1. CLONE CODE ===
        stage('Checkout') {
            steps {
                git branch: 'fix/add-tls-config-redis', 
                     url: 'https://github.com/dnamxtvl/laravel_quizze'
                
                sh """
                    echo "📁 Workspace: ${WORKSPACE}"
                    echo "📂 Kiểm tra file .env.example:"
                    ls -la .env.example || echo "Không tìm thấy .env.example"
                """
            }
        }

        // === 2. TẠO FILE .ENV ===
        stage('Prepare ENV file') {
            steps {
                withCredentials([
                    string(credentialsId: 'APP_NAME', variable: 'APP_NAME'),
                    string(credentialsId: 'APP_KEY', variable: 'APP_KEY'),
                    string(credentialsId: 'APP_URL', variable: 'APP_URL'),
                    string(credentialsId: 'DB_CONNECTION', variable: 'DB_CONNECTION'),
                    string(credentialsId: 'ECS_DB_HOST', variable: 'ECS_DB_HOST'),
                    string(credentialsId: 'DB_PORT', variable: 'DB_PORT'),
                    string(credentialsId: 'ECS_DB_DATABASE', variable: 'ECS_DB_DATABASE'),
                    string(credentialsId: 'ECS_DB_USERNAME', variable: 'ECS_DB_USERNAME'),
                    string(credentialsId: 'ECS_DB_PASSWORD', variable: 'ECS_DB_PASSWORD'),
                    string(credentialsId: 'ECS_REDIS_CLIENT', variable: 'ECS_REDIS_CLIENT'),
                    string(credentialsId: 'ECS_REDIS_HOST', variable: 'ECS_REDIS_HOST'),
                    string(credentialsId: 'ECS_MAIL_MAILER', variable: 'ECS_MAIL_MAILER'),
                    string(credentialsId: 'ECS_MAIL_PORT', variable: 'ECS_MAIL_PORT'),
                    string(credentialsId: 'ECS_MAIL_FROM_ADDRESS', variable: 'ECS_MAIL_FROM_ADDRESS'),
                    
                    // SỬA: Dùng đúng credential IDs cho AWS
                    string(credentialsId: 'ECS_AWS_ACCESS_KEY_ID', variable: 'ECS_AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'ECS_AWS_SECRET_ACCESS_KEY', variable: 'ECS_AWS_SECRET_ACCESS_KEY'),
                    string(credentialsId: 'ECS_AWS_DEFAULT_REGION', variable: 'ECS_AWS_DEFAULT_REGION'),
                    
                    string(credentialsId: 'AWS_URL', variable: 'AWS_URL'),
                    string(credentialsId: 'REVERB_APP_ID', variable: 'REVERB_APP_ID'),
                    string(credentialsId: 'REVERB_APP_KEY', variable: 'REVERB_APP_KEY'),
                    string(credentialsId: 'REVERB_HOST', variable: 'REVERB_HOST'),
                    string(credentialsId: 'REVERB_APP_SECRET', variable: 'REVERB_APP_SECRET'),
                    string(credentialsId: 'FRONT_END_URL', variable: 'FRONT_END_URL'),
                    string(credentialsId: 'GOOGLE_CLIENT_ID', variable: 'GOOGLE_CLIENT_ID'),
                    string(credentialsId: 'GOOGLE_CLIENT_SECRET', variable: 'GOOGLE_CLIENT_SECRET'),
                    string(credentialsId: 'GOOGLE_REDIRECT', variable: 'GOOGLE_REDIRECT'),
                    string(credentialsId: 'MAXMIND_LICENSE_KEY', variable: 'MAXMIND_LICENSE_KEY'),
                    string(credentialsId: 'CLOUDWATCH_LOG_GROUP', variable: 'CLOUDWATCH_LOG_GROUP'),
                    string(credentialsId: 'CLOUDWATCH_LOG_STREAM', variable: 'CLOUDWATCH_LOG_STREAM'),
                    string(credentialsId: 'ECS_LOG_CHANNEL', variable: 'ECS_LOG_CHANNEL'),
                    string(credentialsId: 'ECS_LOG_LEVEL', variable: 'ECS_LOG_LEVEL')
                ]) {
                    sh """
                        echo "📝 Tạo file .env từ .env.example..."
                        cp .env.example .env
                        
                        echo "🔄 Cập nhật các biến môi trường..."
                        # Cấu hình cơ bản
                        sed -i 's|^APP_ENV=.*|APP_ENV=production|' .env
                        sed -i 's|^APP_DEBUG=.*|APP_DEBUG=false|' .env
                        
                        # App config
                        sed -i 's|^APP_NAME=.*|APP_NAME=\"${APP_NAME}\"|' .env
                        sed -i 's|^APP_KEY=.*|APP_KEY=${APP_KEY}|' .env
                        sed -i 's|^APP_URL=.*|APP_URL=${APP_URL}|' .env
                        
                        # Database
                        sed -i 's|^DB_CONNECTION=.*|DB_CONNECTION=${DB_CONNECTION}|' .env
                        sed -i 's|^DB_HOST=.*|DB_HOST=${ECS_DB_HOST}|' .env
                        sed -i 's|^DB_PORT=.*|DB_PORT=${DB_PORT}|' .env
                        sed -i 's|^DB_DATABASE=.*|DB_DATABASE=${ECS_DB_DATABASE}|' .env
                        sed -i 's|^DB_USERNAME=.*|DB_USERNAME=${ECS_DB_USERNAME}|' .env
                        sed -i 's|^DB_PASSWORD=.*|DB_PASSWORD=${ECS_DB_PASSWORD}|' .env
                        
                        # Redis
                        sed -i 's|^REDIS_CLIENT=.*|REDIS_CLIENT=${ECS_REDIS_CLIENT}|' .env
                        sed -i 's|^REDIS_HOST=.*|REDIS_HOST=${ECS_REDIS_HOST}|' .env
                        
                        # Mail
                        sed -i 's|^MAIL_MAILER=.*|MAIL_MAILER=${ECS_MAIL_MAILER}|' .env
                        sed -i 's|^MAIL_PORT=.*|MAIL_PORT=${ECS_MAIL_PORT}|' .env
                        sed -i 's|^MAIL_FROM_ADDRESS=.*|MAIL_FROM_ADDRESS=${ECS_MAIL_FROM_ADDRESS}|' .env
                        
                        # AWS (SỬA: dùng biến từ credentials ECS)
                        sed -i 's|^AWS_ACCESS_KEY_ID=.*|AWS_ACCESS_KEY_ID=${ECS_AWS_ACCESS_KEY_ID}|' .env
                        sed -i 's|^AWS_SECRET_ACCESS_KEY=.*|AWS_SECRET_ACCESS_KEY=${ECS_AWS_SECRET_ACCESS_KEY}|' .env
                        sed -i 's|^AWS_DEFAULT_REGION=.*|AWS_DEFAULT_REGION=${ECS_AWS_DEFAULT_REGION}|' .env
                        sed -i 's|^AWS_URL=.*|AWS_URL=${AWS_URL}|' .env
                        
                        # Reverb (WebSocket)
                        sed -i 's|^REVERB_APP_ID=.*|REVERB_APP_ID=${REVERB_APP_ID}|' .env
                        sed -i 's|^REVERB_APP_KEY=.*|REVERB_APP_KEY=${REVERB_APP_KEY}|' .env
                        sed -i 's|^REVERB_HOST=.*|REVERB_HOST=${REVERB_HOST}|' .env
                        sed -i 's|^REVERB_APP_SECRET=.*|REVERB_APP_SECRET=${REVERB_APP_SECRET}|' .env
                        
                        # Frontend
                        sed -i 's|^FRONT_END_URL=.*|FRONT_END_URL=${FRONT_END_URL}|' .env
                        
                        # Google OAuth
                        sed -i 's|^GOOGLE_CLIENT_ID=.*|GOOGLE_CLIENT_ID=${GOOGLE_CLIENT_ID}|' .env
                        sed -i 's|^GOOGLE_CLIENT_SECRET=.*|GOOGLE_CLIENT_SECRET=${GOOGLE_CLIENT_SECRET}|' .env
                        sed -i 's|^GOOGLE_REDIRECT=.*|GOOGLE_REDIRECT=${GOOGLE_REDIRECT}|' .env
                        
                        # MaxMind
                        sed -i 's|^MAXMIND_LICENSE_KEY=.*|MAXMIND_LICENSE_KEY=${MAXMIND_LICENSE_KEY}|' .env
                        
                        # CloudWatch Logging
                        sed -i 's|^CLOUDWATCH_LOG_GROUP=.*|CLOUDWATCH_LOG_GROUP=${CLOUDWATCH_LOG_GROUP}|' .env
                        sed -i 's|^CLOUDWATCH_LOG_STREAM=.*|CLOUDWATCH_LOG_STREAM=${CLOUDWATCH_LOG_STREAM}|' .env
                        sed -i 's|^LOG_CHANNEL=.*|LOG_CHANNEL=${ECS_LOG_CHANNEL}|' .env
                        sed -i 's|^LOG_STACK=.*|LOG_STACK=${ECS_LOG_CHANNEL}|' .env
                        sed -i 's|^LOG_LEVEL=.*|LOG_LEVEL=${ECS_LOG_LEVEL}|' .env
                        
                        # Firebase
                        sed -i 's|^FIREBASE_CREDENTIALS=.*|FIREBASE_CREDENTIALS=/var/www/html/storage/app/firebase/service-account.json|' .env
                        
                        echo "✅ File .env đã được tạo"
                    """
                }
            }
        }

        // === 3. COPY FIREBASE CREDENTIALS ===
        stage('Copy Firebase Credentials') {
            steps {
                withCredentials([file(credentialsId: 'FIREBASE_CREDENTIALS', variable: 'FIREBASE_FILE')]) {
                    sh """
                        echo "🔥 Copy Firebase credentials..."
                        mkdir -p storage/app/firebase
                        cp "${FIREBASE_FILE}" "storage/app/firebase/service-account.json"
                        chmod 644 storage/app/firebase/service-account.json
                        
                        echo "📁 Kiểm tra file Firebase:"
                        ls -la storage/app/firebase/
                    """
                }
            }
        }

        // === 4. LOGIN AWS ECR (SỬA LẠI) ===
        stage('Login to AWS ECR') {
            steps {
                // Dùng đúng credential IDs: ECS_AWS_ACCESS_KEY_ID và ECS_AWS_SECRET_ACCESS_KEY
                withCredentials([
                    string(credentialsId: 'ECS_AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'ECS_AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh """
                        echo "🔐 Đang login vào AWS ECR..."
                        echo "AWS Region: ${AWS_REGION}"
                        echo "ECR Registry: 147925596024.dkr.ecr.ap-southeast-1.amazonaws.com"
                        
                        # Set AWS credentials từ Jenkins credentials
                        export AWS_ACCESS_KEY_ID="\${AWS_ACCESS_KEY_ID}"
                        export AWS_SECRET_ACCESS_KEY="\${AWS_SECRET_ACCESS_KEY}"
                        export AWS_DEFAULT_REGION="${AWS_REGION}"
                        
                        # Debug: kiểm tra biến môi trường (không in secret)
                        echo "AWS_ACCESS_KEY_ID length: \${#AWS_ACCESS_KEY_ID}"
                        echo "AWS_SECRET_ACCESS_KEY length: \${#AWS_SECRET_ACCESS_KEY}"
                        
                        # Test AWS CLI
                        echo "🔍 Kiểm tra AWS CLI..."
                        aws sts get-caller-identity --region ${AWS_REGION} --output text --query 'Account' || echo "❌ Không thể xác thực AWS"
                        
                        # Login vào ECR
                        echo "🔐 Login vào ECR..."
                        aws ecr get-login-password --region ${AWS_REGION} \\
                            | docker login --username AWS --password-stdin 147925596024.dkr.ecr.ap-southeast-1.amazonaws.com
                        
                        if [ \$? -eq 0 ]; then
                            echo "✅ Login ECR thành công"
                        else
                            echo "❌ Login ECR thất bại"
                            exit 1
                        fi
                    """
                }
            }
        }
        
        // === 4. FIX DOCKERIGNORE ===
        stage('Prepare for Docker Build') {
            steps {
                sh """
                    echo "🛠️ Chuẩn bị cho Docker build..."
                    
                    # Kiểm tra và sửa .dockerignore
                    if [ -f ".dockerignore" ]; then
                        echo "📄 Tìm thấy .dockerignore"
                        echo "=== Nội dung hiện tại ==="
                        cat .dockerignore
                        
                        # Sử dụng sed đơn giản không cần regex phức tạp
                        # Xóa tất cả dòng chứa .env
                        sed -i '/\\.env/d' .dockerignore
                        
                        # Xóa dòng chứa *.env  
                        sed -i '/\\*\\.env/d' .dockerignore
                        
                        echo "=== Nội dung sau khi sửa ==="
                        cat .dockerignore
                    else
                        echo "ℹ️ Không có file .dockerignore"
                    fi
                    
                    # Đảm bảo .env tồn tại
                    if [ ! -f ".env" ]; then
                        echo "❌ Không tìm thấy file .env"
                        exit 1
                    fi
                    
                    echo "✅ Sẵn sàng cho Docker build"
                """
            }
        }

        // === 5. BUILD DOCKER IMAGES ===
        stage('Build Docker Images') {
            steps {
                sh """
                    echo "🐳 Building PHP-FPM image với tag: ${IMAGE_TAG}"
                    docker build -t ${REPO_PHP}:${IMAGE_TAG} -t ${REPO_PHP}:latest -f docker/Dockerfile.production .
                    
                    echo "🐳 Building Nginx image với tag: ${IMAGE_TAG}"
                    docker build -t ${REPO_NGINX}:${IMAGE_TAG} -t ${REPO_NGINX}:latest -f docker/Dockerfile.nginx.production .
                    
                    echo "📦 Images built:"
                    docker images | grep -E '(${REPO_PHP}|${REPO_NGINX})'
                """
            }
        }
        
        // === 6. PUSH IMAGES TO ECR ===
        stage('Push Images to AWS ECR') {
            steps {
                sh """
                    echo "🚀 Pushing PHP-FPM images to ECR..."
                    docker push ${REPO_PHP}:${IMAGE_TAG}
                    docker push ${REPO_PHP}:latest
                    
                    echo "🚀 Pushing Nginx images to ECR..."
                    docker push ${REPO_NGINX}:${IMAGE_TAG}
                    docker push ${REPO_NGINX}:latest
                    
                    echo "✅ Images đã được push lên ECR với tag: ${IMAGE_TAG}"
                """
            }
        }

        // === 7. FORCE DEPLOY TO ECS
        stage('Deploy to ECS') {
            steps {
                echo "🔄 Cập nhật ECS service..."
                script {
                    sh "aws ecs update-service --cluster quiz-ecs-cluster --service quiz-ecs-service --force-new-deployment"
                    echo "📋 Force deploy completed"
                }
            }
        }
    }

    post {
        always {
            echo "🧹 Cleaning up..."
            
            sh """
                echo "🗑️ Xóa images cục bộ của build này..."
                docker rmi ${REPO_PHP}:${IMAGE_TAG} 2>/dev/null || true
                docker rmi ${REPO_NGINX}:${IMAGE_TAG} 2>/dev/null || true
                
                echo "✅ Cleanup hoàn tất"
            """
        }
        
        success {
            echo "🎉 DEPLOY THÀNH CÔNG! Image tag: ${IMAGE_TAG}"
        }
        
        failure {
            echo "❌ DEPLOY THẤT BẠI!"
        }
    }
}