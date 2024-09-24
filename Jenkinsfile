pipeline{
    agent any

    environment{
        NETLIFY_SITE_ID = 'd2ad0274-b7df-4a3b-b193-64c2a07dab69'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages{
        stage('build'){
        agent {
            docker{
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

        stage('tests'){
            parallel{
                stage('unit test'){
                agent {
                  docker{
                    image 'node:18-alpine'
                    reuseNode true
                  }
                }

                    steps{
                        sh '''
                            test -f build/index.html
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }
                stage('E2E'){
                    agent {
                        docker{
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }

                    steps{
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }

            }
        }

        stage('deploy'){
            agent {
                docker{
                    image 'node:18-alpine'
                    reuseNode true
            }
        }
            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. Site ID: &NETLIFY_SITE_ID "
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                    


                '''
            }
        }

        stage('prod E2E'){
                    agent {
                        docker{
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }

                    environment{
                        CI_ENVIRONMENT_URL = 'https://steady-croissant-00dd27.netlify.app'
    }

                    steps{
                        sh '''
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
    }
    
}
