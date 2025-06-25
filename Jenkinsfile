pipeline {
    agent {
        docker { image 'node:20-alpine' }
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build Application') {
            steps {
                sh 'echo "Node.js Version:"'
                sh 'node --version'
                sh 'echo "npm Version:"'
                sh 'npm --version'
                sh 'npm ci'
                sh 'npm run build'
            }
        }
    }
}