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
                # Ensure the network exists for communication 
                docker network create app-network || true
                
                # Force remove old instances to prevent name conflicts [cite: 107-112]
                docker rm -f backend1 backend2 || true
                
                # Start both containers on the shared network
                docker run -d --name backend1 --network app-network backend-app
                docker run -d --name backend2 --network app-network backend-app
                '''
            }
        }
        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                docker rm -f nginx-lb || true
                
                # Run NGINX on the same network [cite: 154-159]
                docker run -d \
                  --name nginx-lb \
                  --network app-network \
                  -p 80:80 \
                  nginx
                
                # Copy the configuration file into the container [cite: 184]
                docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf
                
                # Reload NGINX to apply the load balancing strategy [cite: 887]
                docker exec nginx-lb nginx -s reload
                '''
            }
        }
    }
    post {
        success {
            echo 'Pipeline executed successfully. NGINX load balancer is running.' [cite: 656-657]
        }
        failure {
            echo 'Pipeline failed. Check console logs for errors.' [cite: 683]
        }
    }
}
