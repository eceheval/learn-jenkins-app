pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                echo 'Building...'
                sh 'docker build -t myapp .'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing...'
                sh 'docker run --rm myapp test'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying...'
                sh 'docker run -d -p 8080:80 myapp'
            }
        }
    }
    
    post {
        always {
            echo 'Cleaning up...'
            sh 'docker system prune -f'
        }
    }
}
