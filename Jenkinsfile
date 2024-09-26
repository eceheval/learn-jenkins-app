pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '22cc4b5c-6923-49b2-87fc-e8c9a5f922d9'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        DOCKER_IMAGE = 'eceunal7/eceunal7deneme'
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_CREDENTIALS_ID = 'dockerhub_credentials'  
    }

    stages {
        stage('Build') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                echo "Build started..."
                ls -la
                node --version
                npm --version
                npm ci
                npm run build
                ls -la
                '''
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${env.BUILD_NUMBER}")
                }
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS_ID}") {
                        docker.image("${DOCKER_IMAGE}:${env.BUILD_NUMBER}").push()
                    }
                }   
             }
        }


        stage('Tests'){
            parallel {
                stage('Unit Tests'){

                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }


                    steps {
                        sh '''
                        echo "Testler başlatılıyor.."

                        echo "Index dosyası kontrol ediliyor.."

                        if ls build/index.html; then
                              echo "index.html var"
                        else
                              echo "index.html yok"
                              exit 1
                        fi
                        npm test
                        '''
                    
                    }


                    post {
                        always{
                            junit 'jest-results/junit.xml'
                        }
                    }

                }


                stage('E2E'){
                    agent{
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
                        always{
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }

                
            }
        }

                stage('Deploy to Docker') {
                    steps {
                        script {
                            def imageName = "${DOCKER_IMAGE}:${env.BUILD_NUMBER}"
                            sh '''
                            docker run -d -p 3000:3000 ''' + imageName + '''
                            '''
                            echo "Application successfully deployed."
                        }
                    }
                }

                stage('Run Docker Container Locally') {
            steps {
                script {
                    // Portun kullanımda olup olmadığını kontrol et
                    def isPortInUse = sh(script: 'lsof -i :3000', returnStatus: true) == 0
                    
                    if (isPortInUse) {
                        echo "3000 portu zaten kullanımda. Farklı bir port kullanılacak."
                        // Farklı bir port numarası belirle
                        def newPort = 3001
                        // Docker container'ı yeni port ile çalıştır
                        sh '''
                        docker run -d -p ${newPort}:3000 ${DOCKER_IMAGE}:${env.BUILD_NUMBER}
                        '''
                        echo "Yerel Docker container ${newPort} portu ile başarıyla çalıştırıldı."
                    } else {
                        // Port kullanılmıyorsa, 3000 portu ile çalıştır
                        sh '''
                        docker run -d -p 3000:3000 ${DOCKER_IMAGE}:${env.BUILD_NUMBER}
                        '''
                        echo "Yerel Docker container başarıyla 3000 portu ile çalıştırıldı."
                    }
                }
            }
        }


        stage('Check Application') {
            steps {
                script {
                    sh '''
                    # Yerel container'ın çalışıp çalışmadığını kontrol etmek için
                    curl http://localhost:3000
                    '''
                }
            }
        }
        

        stage('Deploy to Netlify') {
            agent{
                docker{
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
        
    }
}