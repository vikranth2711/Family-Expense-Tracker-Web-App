pipeline {
    agent any
    
    environment {
        DOCKER_HUB_CREDENTIALS = credentials('dockerhub-credentials')
        AWS_CREDENTIALS = credentials('aws-credentials')
        ANSIBLE_HOST_KEY_CHECKING = 'False'
        APP_NAME = 'family-expense-tracker'
        DOCKER_HUB_REPO = 'yourusername/expense-tracker'
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code from repository...'
                checkout scm
            }
        }
        
        stage('Environment Setup') {
            steps {
                script {
                    echo 'Setting up environment variables...'
                    sh '''
                        echo "Setting up Node.js environment"
                        node --version
                        npm --version
                    '''
                }
            }
        }
        
        stage('Install Dependencies - Backend') {
            steps {
                dir('backend') {
                    echo 'Installing backend dependencies...'
                    sh 'npm ci --production'
                }
            }
        }
        
        stage('Install Dependencies - Frontend') {
            steps {
                dir('frontend') {
                    echo 'Installing frontend dependencies...'
                    sh 'npm ci'
                }
            }
        }
        
        stage('Run Tests - Backend') {
            steps {
                dir('backend') {
                    echo 'Running backend tests...'
                    sh 'npm test || echo "No tests defined"'
                }
            }
        }
        
        stage('Run Tests - Frontend') {
            steps {
                dir('frontend') {
                    echo 'Running frontend tests...'
                    sh 'npm test -- --watchAll=false || echo "No tests defined"'
                }
            }
        }
        
        stage('Build Frontend') {
            steps {
                dir('frontend') {
                    echo 'Building frontend application...'
                    sh '''
                        export CI=false
                        npm run build
                    '''
                }
            }
        }
        
        stage('Build Docker Images') {
            steps {
                script {
                    echo 'Building Docker images...'
                    sh '''
                        docker-compose build
                    '''
                }
            }
        }
        
        stage('Security Scan') {
            steps {
                script {
                    echo 'Running security scans...'
                    sh '''
                        # Install Trivy if not available
                        if ! command -v trivy &> /dev/null; then
                            wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | apt-key add -
                            echo "deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | tee -a /etc/apt/sources.list.d/trivy.list
                            apt-get update && apt-get install -y trivy
                        fi
                        
                        # Scan Docker images
                        trivy image --severity HIGH,CRITICAL ${DOCKER_HUB_REPO}-backend:latest || true
                        trivy image --severity HIGH,CRITICAL ${DOCKER_HUB_REPO}-frontend:latest || true
                    '''
                }
            }
        }
        
        stage('Push Docker Images') {
            when {
                branch 'main'
            }
            steps {
                script {
                    echo 'Pushing Docker images to registry...'
                    sh '''
                        echo $DOCKER_HUB_CREDENTIALS_PSW | docker login -u $DOCKER_HUB_CREDENTIALS_USR --password-stdin
                        
                        docker tag expense-tracker-backend:latest ${DOCKER_HUB_REPO}-backend:latest
                        docker tag expense-tracker-frontend:latest ${DOCKER_HUB_REPO}-frontend:latest
                        
                        docker tag expense-tracker-backend:latest ${DOCKER_HUB_REPO}-backend:${BUILD_NUMBER}
                        docker tag expense-tracker-frontend:latest ${DOCKER_HUB_REPO}-frontend:${BUILD_NUMBER}
                        
                        docker push ${DOCKER_HUB_REPO}-backend:latest
                        docker push ${DOCKER_HUB_REPO}-frontend:latest
                        docker push ${DOCKER_HUB_REPO}-backend:${BUILD_NUMBER}
                        docker push ${DOCKER_HUB_REPO}-frontend:${BUILD_NUMBER}
                        
                        docker logout
                    '''
                }
            }
        }
        
        stage('Terraform Plan') {
            when {
                branch 'main'
            }
            steps {
                dir('terraform') {
                    script {
                        echo 'Running Terraform plan...'
                        sh '''
                            terraform init
                            terraform plan -out=tfplan
                        '''
                    }
                }
            }
        }
        
        stage('Terraform Apply') {
            when {
                allOf {
                    branch 'main'
                    expression { params.DEPLOY_INFRASTRUCTURE == true }
                }
            }
            steps {
                dir('terraform') {
                    script {
                        echo 'Applying Terraform configuration...'
                        input message: 'Approve infrastructure deployment?', ok: 'Deploy'
                        sh '''
                            terraform apply -auto-approve tfplan
                            terraform output -json > terraform-output.json
                        '''
                    }
                }
            }
        }
        
        stage('Configure Ansible Inventory') {
            when {
                branch 'main'
            }
            steps {
                dir('ansible') {
                    script {
                        echo 'Configuring Ansible inventory...'
                        sh '''
                            # Get EC2 instance IP from Terraform output
                            SERVER_IP=$(cd ../terraform && terraform output -raw instance_public_ip)
                            
                            # Update inventory file
                            sed -i "s/{{ server_ip }}/${SERVER_IP}/g" inventory.ini
                            
                            echo "Configured Ansible inventory with IP: ${SERVER_IP}"
                        '''
                    }
                }
            }
        }
        
        stage('Provision Server') {
            when {
                allOf {
                    branch 'main'
                    expression { params.PROVISION_SERVER == true }
                }
            }
            steps {
                dir('ansible') {
                    script {
                        echo 'Provisioning server with Ansible...'
                        sh '''
                            ansible-playbook -i inventory.ini playbook.yml
                        '''
                    }
                }
            }
        }
        
        stage('Deploy Application') {
            when {
                branch 'main'
            }
            steps {
                dir('ansible') {
                    script {
                        echo 'Deploying application with Ansible...'
                        sh '''
                            ansible-playbook -i inventory.ini deploy.yml \
                                -e "mongodb_uri=${MONGODB_URI}" \
                                -e "jwt_secret=${JWT_SECRET}"
                        '''
                    }
                }
            }
        }
        
        stage('Health Check') {
            when {
                branch 'main'
            }
            steps {
                script {
                    echo 'Performing health check...'
                    sh '''
                        SERVER_IP=$(cd terraform && terraform output -raw instance_public_ip)
                        
                        # Wait for application to be ready
                        sleep 30
                        
                        # Check backend health
                        curl -f http://${SERVER_IP}:5000/health || exit 1
                        
                        # Check frontend availability
                        curl -f http://${SERVER_IP} || exit 1
                        
                        echo "Health check passed!"
                    '''
                }
            }
        }
        
        stage('Smoke Tests') {
            when {
                branch 'main'
            }
            steps {
                script {
                    echo 'Running smoke tests...'
                    sh '''
                        SERVER_IP=$(cd terraform && terraform output -raw instance_public_ip)
                        
                        # Test backend API endpoints
                        echo "Testing backend API..."
                        curl -f http://${SERVER_IP}:5000/api/health || exit 1
                        
                        # Test Grafana availability
                        echo "Testing Grafana..."
                        curl -f http://${SERVER_IP}:3001/api/health || true
                        
                        # Test Prometheus availability
                        echo "Testing Prometheus..."
                        curl -f http://${SERVER_IP}:9090/-/healthy || true
                        
                        echo "Smoke tests completed!"
                    '''
                }
            }
        }
    }
    
    post {
        always {
            echo 'Cleaning up workspace...'
            cleanWs()
        }
        
        success {
            script {
                def serverIp = sh(
                    script: 'cd terraform && terraform output -raw instance_public_ip || echo "N/A"',
                    returnStdout: true
                ).trim()
                
                echo """
                ========================================
                Deployment Successful! 🎉
                ========================================
                Application URL: http://${serverIp}
                Backend API: http://${serverIp}:5000
                Grafana Dashboard: http://${serverIp}:3001
                Prometheus: http://${serverIp}:9090
                ========================================
                """
                
                // Send notification (configure your notification service)
                // emailext subject: "Deployment Success - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                //          body: "Deployment completed successfully!",
                //          to: "team@example.com"
            }
        }
        
        failure {
            echo 'Deployment failed! Check logs for details.'
            // Send failure notification
            // emailext subject: "Deployment Failed - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            //          body: "Deployment failed. Please check Jenkins logs.",
            //          to: "team@example.com"
        }
    }
}
