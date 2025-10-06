pipeline {
    environment {
        NETLIFY_SITE_ID = '5e83c00f-7843-498e-8cd3-42bad2b03b12'
    }

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
                '''
            }
        }
    
        stage('Run Tests') {
            parallel {
                stage('Unit Test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            test -f build/index.html
                            npm test
                        '''
                    }
                }
                  
                stage('Staging E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.54.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            npm install serve
                            # Start static server on port 3000 because Playwright's baseURL uses http://localhost:3000
                            node_modules/.bin/serve -s build -l 3000 &
                            sleep 5
                            npx playwright install
                            npx playwright test
                        '''
                    }
                    post {
                        always {
                        echo 'E2E tests completed'
                        }
                    }
                }
            }
        }

        stage('Deploy Prod') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                withCredentials([string(credentialsId: 'netlify-token', variable: 'NETLIFY_AUTH_TOKEN')]) {
                    sh '''
                        npm install netlify-cli node-jq
                        node_modules/.bin/netlify --version
                        echo "Deploying to production/ Site (Project) ID: $NETLIFY_SITE_ID"
                        
                        echo "show netlify status"
                        node_modules/.bin/netlify status
                        
                        echo " ***** run build ***** "
                        node_modules/.bin/netlify deploy \
                            --dir=build \
                            --prod \
                            --json \
                            --site=$NETLIFY_SITE_ID \
                            --auth=$NETLIFY_AUTH_TOKEN > deploy-output.json
                        
                        node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json
                        
                        echo " ***** after build ***** "
                    '''
                    script {
                        env.STAGING_URL = sh(script: "node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json", returnStdout: true).trim()
                    }
                }
            }
        }

        stage('Prod E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.54.0-jammy'
                    reuseNode true
                }
            }
            environment {
                // Export the staging URL into CI_ENVIRONMENT_URL so Playwright's baseURL picks it up
                CI_ENVIRONMENT_URL = "${env.STAGING_URL}"
            }
            steps {
                sh '''
                    node --version
                    
                    echo "Running post-deployment tests..."
                    # Ensure playwright browsers are installed for this image/version
                                        echo "CI_ENVIRONMENT_URL=$CI_ENVIRONMENT_URL"
                                        # Try a quick HEAD request to validate the staging URL is reachable.
                                        # Use curl if available, otherwise use a small node script.
                                        if command -v curl >/dev/null 2>&1; then
                                            curl -I "$CI_ENVIRONMENT_URL" || true
                                        else
                                            node -e "const https = require('https'); const u=process.env.CI_ENVIRONMENT_URL; if(!u){console.error('CI_ENVIRONMENT_URL not set'); process.exit(0);} https.get(u, r=>{console.log('status', r.statusCode); r.resume();}).on('error', e=>{console.error('connect error', e.message)});"
                                        fi
                                        npx playwright install
                                        npx playwright test --reporter=html
                '''
            }
        }
    }

    post {
        always {
            junit 'jest-results/junit.xml'
        }
    }
}