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
                        node_modules/.bin/netlify --version
                        echo "Deploying to production/ Site (Project) ID: $NETLIFY_SITE_ID"
                        

                        echo "show netlify status"
                        node_modules/.bin/netlify status
                        
                        echo " ***** run build ***** "
                        node_modules/.bin/netlify deploy \
                            --dir=build \
                            --prod \
                            --site=$NETLIFY_SITE_ID \
                            --auth=$NETLIFY_AUTH_TOKEN"
                        

                        echo " ***** after build ***** "
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

                        // # Deploy without triggering Netlify's build process
                        // node_modules/.bin/netlify deploy \
                        //     --prod \
                        //     --dir=build \
                        //     --site=$NETLIFY_SITE_ID \
                        //     --auth=$NETLIFY_AUTH_TOKEN \
                        //     --build=false
