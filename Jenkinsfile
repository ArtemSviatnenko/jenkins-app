pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = "025acded-9a6d-4ed2-aa60-cb1245ea7c7a"
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

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

        stage ('Tests') {
            parallel {
                stage('Unit Tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }    
                    steps {
                        sh '''
                            echo 'Test stage'
                            # test -f build/index.html || { echo "Error: build/index.html not found!"; exit 1; }
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }    
                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                    
                }
            }
        }

        stage('Deploy staging') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    # Create a local package directory
                    export PATH=$HOME/.local/bin:$PATH
                    mkdir -p $HOME/.local

                    # Install Python and build tools in user space
                    python3 -m venv $HOME/.local/venv
                    source $HOME/.local/venv/bin/activate

                    # Install Netlify CLI and node-jq in user space
                    npm install --prefix $HOME/.local netlify-cli node-jq

                    # Use the locally installed binaries
                    $HOME/.local/node_modules/.bin/netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    $HOME/.local/node_modules/.bin/netlify status
                    $HOME/.local/node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                    $HOME/.local/node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json
                '''
            }
        }

        stage('Approval') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure I want to deploy'
                }
            }
        }

        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }

        stage('Prod E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            environment {
                NETLIFY_SITE_ID = "025acded-9a6d-4ed2-aa60-cb1245ea7c7a"
                NETLIFY_AUTH_TOKEN = credentials('netlify-token')
                CI_ENVIRONMENT_URL = 'https://sensational-entremet-137549.netlify.app'
            }

            steps {
                sh '''
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    junit 'jest-results/junit.xml'
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
}
