pipeline {
    agent any
    stages {
        stage('Build Backend Image') {
            steps {
                sh '''
                docker rmi -f backend-app || true
                docker build -t backend-app backend
                '''
            }
        }
        stage('Deploy Backend Containers') {
            steps {
                sh '''
                docker network create app-network || true
                docker rm -f backend1 backend2 || true
                docker run -d --name backend1 --network app-network backend-app
                docker run -d --name backend2 --network app-network backend-app
                '''
            }
        }
        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                docker rm -f nginx-lb || true
                
                # Start NGINX without the problematic volume mount
                docker run -d --name nginx-lb --network app-network -p 80:80 nginx
                
                # Wait briefly for the container to initialize
                sleep 2
                
                # Copy the local config file into the container 
                docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf
                
                # Reload NGINX to apply the Round-Robin settings
                docker exec nginx-lb nginx -s reload
                '''
            }
        }
    }
    post {
        success {
            echo 'Pipeline executed successfully. NGINX load balancer is running.' 
        }
        failure {
            echo 'Pipeline failed. Check console logs for errors.'
        }
    }
}
