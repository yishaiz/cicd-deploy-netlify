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

                /*
                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.54.0-noble'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 5
                            npx playwright install
                            npx playwright test
                        '''
                    }
                    post {
                        always {
                        echo 'E2E tests completed'
                            // sh 'pkill -f serve || true'
                        }
                    }
                }
                */
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
                withCredentials([string(credentialsId: 'netlify-token', variable: 'NETLIFY_AUTH_TOKEN')]) {
                    sh '''
                        npm install netlify-cli
                        npm install nodemon
                        node_modules/.bin/netlify --version
                        echo "Deploying to production/ Site (Project) ID: $NETLIFY_SITE_ID"
                        
                        # DEBUG - בדיקת הטוקן
                        echo "Token exists: $([ -n "$NETLIFY_AUTH_TOKEN" ] && echo "YES" || echo "NO")"
                        echo "Token length: ${#NETLIFY_AUTH_TOKEN}"
                        echo "Token first 10 chars: ${NETLIFY_AUTH_TOKEN:0:10}..."
                        
                        # ניסיון לאמת עם הטוקן
                        echo "Trying to authenticate..."
                        node_modules/.bin/netlify status --auth=$NETLIFY_AUTH_TOKEN
                        
                        # אם זה עובד, נעשה deploy
                        node_modules/.bin/netlify deploy --prod --dir=build --site=$NETLIFY_SITE_ID --auth=$NETLIFY_AUTH_TOKEN
                    '''
                }
            }
        }
    }

    post {
        always {
            junit 'jest-results/junit.xml'
        }
    }
}