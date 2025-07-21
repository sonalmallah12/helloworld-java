pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'sonalmallah12/hello-java'
        SONAR_HOST_URL = 'http://192.168.1.182:9000'
    }

    stages {
        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        mvn sonar:sonar \
                        -Dsonar.projectKey=JavaApp \
                        -Dsonar.host.url=http://192.168.1.182:9000
                    '''
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker build -t $DOCKER_IMAGE:latest -f ./Dockerfile .
                        docker push $DOCKER_IMAGE:latest
                    '''
                }
            }
        }

        stage('Deploy to K3s') {
            steps {
                sh '''
                    export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
                    kubectl apply -f k8s/deployment.yaml
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Build and Deployment succeeded"
        }
        failure {
            echo "❌ Build or Deployment failed"
        }
    }
}

