pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Test stage') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }    
            steps {
                sh '''
                    ls -la
                    node --version
                    echo 'Test stage'
                    test -f build/index.html || { echo "Error: build/index.html not found!"; exit 1; }
                    npm test
                '''
            }
        }
    }
}
