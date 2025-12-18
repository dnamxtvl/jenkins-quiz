pipeline {
    agent any

    environment {
        GITHUB_REPO    = 'https://github.com/dnamxtvl/nuxt-quizze'
        GITHUB_BRANCH  = 'master'
        
        CODECOMMIT_REPO = 'quizz-app'
        AWS_REGION     = 'ap-southeast-1'
        
        // Temp directory for cloning
        WORKING_DIR    = 'sync_quiz_repo'
    }

    stages {
        // === 1. CLEAN WORKSPACE ===
        stage('Clean Workspace') {
            steps {
                cleanWs()
                sh '''
                    echo "🧹 Cleaning workspace..."
                    rm -rf ${WORKING_DIR} 2>/dev/null || true
                '''
            }
        }

        // === 2. CLONE FROM GITHUB ===
        stage('Clone from GitHub') {
            steps {
                sh """
                    echo "📥 Cloning from GitHub..."
                    echo "Repository: ${GITHUB_REPO}"
                    echo "Branch: ${GITHUB_BRANCH}"
                    
                    git clone --branch ${GITHUB_BRANCH} ${GITHUB_REPO} ${WORKING_DIR}
                    
                    cd ${WORKING_DIR}
                    echo "✅ GitHub repo cloned"
                    echo "📁 Directory structure:"
                    pwd
                    ls -la
                    echo "🔗 Git remote:"
                    git remote -v
                    echo "📋 Git branches:"
                    git branch -a
                """
            }
        }

        // === 3. LOGIN AWS
        stage('Login to AWS') {
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
                        echo "🔐 Login vào AWS..."
                        aws ecr get-login-password --region ${AWS_REGION} \\
                            | docker login --username AWS --password-stdin 147925596024.dkr.ecr.ap-southeast-1.amazonaws.com
                        
                        if [ \$? -eq 0 ]; then
                            echo "✅ Login Ehành công"
                        else
                            echo "❌ Login Ehất bại"
                            exit 1
                        fi
                    """
                }
            }
        }

        // === 4. CONFIGURE GIT FOR CODECOMMIT ===
        stage('Configure Git for CodeCommit') {
            steps {
                sh """
                    echo "⚙️ Configuring Git for CodeCommit..."
                    cd ${WORKING_DIR}
                    
                    # Cấu hình git user
                    git config --global user.name "Jenkins Sync"
                    git config --global user.email "jenkins@example.com"
                    
                    # Cấu hình credential helper cho CodeCommit
                    git config --global credential.helper '!aws codecommit credential-helper \$@'
                    git config --global credential.UseHttpPath true
                    
                    echo "✅ Git configured for CodeCommit"
                """
            }
        }

        // === 5. VERIFY/CREATE CODECOMMIT REPOSITORY ===
        stage('Verify CodeCommit Repository') {
            steps {
                withCredentials([
                    string(credentialsId: 'ECS_AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'ECS_AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh """
                        echo "🔍 Verifying CodeCommit repository..."
                        export AWS_ACCESS_KEY_ID="${AWS_ACCESS_KEY_ID}"
                        export AWS_SECRET_ACCESS_KEY="${AWS_SECRET_ACCESS_KEY}"
                        export AWS_DEFAULT_REGION="${AWS_REGION}"
                        
                        # Kiểm tra repository tồn tại
                        if aws codecommit get-repository --repository-name ${CODECOMMIT_REPO} 2>/dev/null; then
                            echo "✅ CodeCommit repository '${CODECOMMIT_REPO}' exists"
                        else
                            echo "📦 Creating CodeCommit repository '${CODECOMMIT_REPO}'..."
                            aws codecommit create-repository \\
                                --repository-name ${CODECOMMIT_REPO} \\
                                --repository-description "Synced from GitHub: ${GITHUB_REPO}"
                            
                            if [ \$? -eq 0 ]; then
                                echo "✅ Repository created successfully"
                            else
                                echo "❌ Failed to create repository"
                                exit 1
                            fi
                        fi
                    """
                }
            }
        }

        // === 6. PUSH TO CODECOMMIT ===
        stage('Push to CodeCommit') {
            steps {
                withCredentials([
                    string(credentialsId: 'ECS_AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'ECS_AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh """
                        export AWS_ACCESS_KEY_ID="${AWS_ACCESS_KEY_ID}"
                        export AWS_SECRET_ACCESS_KEY="${AWS_SECRET_ACCESS_KEY}"
                        export AWS_DEFAULT_REGION="${AWS_REGION}"
        
                        cd ${WORKING_DIR}
        
                        # Set remote HTTPS KHÔNG CÓ credentials
                        CODECOMMIT_URL="https://git-codecommit.${AWS_REGION}.amazonaws.com/v1/repos/${CODECOMMIT_REPO}"
        
                        if git remote | grep -q codecommit; then
                            git remote set-url codecommit "\${CODECOMMIT_URL}"
                        else
                            git remote add codecommit "\${CODECOMMIT_URL}"
                        fi
        
                        # Push
                        git push codecommit ${GITHUB_BRANCH}:master --force
                    """
                }
            }
        }
        
        // === 7. VERIFY SYNC ===
        stage('Verify Sync') {
            steps {
                withCredentials([
                    string(credentialsId: 'ECS_AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'ECS_AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh """
                        echo "🔍 Verifying sync..."
                        export AWS_ACCESS_KEY_ID="${AWS_ACCESS_KEY_ID}"
                        export AWS_SECRET_ACCESS_KEY="${AWS_SECRET_ACCESS_KEY}"
                        export AWS_DEFAULT_REGION="${AWS_REGION}"
                        
                        # Lấy thông tin repository
                        echo "📋 CodeCommit repository information:"
                        aws codecommit get-repository --repository-name ${CODECOMMIT_REPO} \\
                            --query 'repositoryMetadata.[repositoryName, repositoryId, cloneUrlHttp]' \\
                            --output table
                        
                        # Lấy commit history mới nhất
                        echo "📜 Latest commits in CodeCommit:"
                        cd ${WORKING_DIR}
                        git fetch codecommit
                        git log codecommit/master --oneline -5 2>/dev/null || echo "ℹ️ Could not fetch commit history"
                        
                        echo "✅ Sync verification completed!"
                    """
                }
            }
        }
    }

    post {
        always {
            echo "🧹 Cleaning up..."
            sh '''
                echo "🗑️ Removing temporary directory..."
                rm -rf ${WORKING_DIR} 2>/dev/null || true
            '''
        }
        success {
            echo "🎉 SYNC THÀNH CÔNG!"
            echo "✅ Code đã được đẩy từ:"
            echo "   GitHub: ${GITHUB_REPO} (branch: ${GITHUB_BRANCH})"
            echo "   → CodeCommit: ${CODECOMMIT_REPO} (branch: master)"
        }
        failure {
            echo "❌ SYNC THẤT BẠI!"
        }
    }
}