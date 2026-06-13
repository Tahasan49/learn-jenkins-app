pipeline {
    agent any

    stages {
        /*
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
        */

        stage('Test') {
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
                            ls -la

                            if [ -f build/index.html ]; then
                                echo "build/index.html var"
                            else
                                echo "build/index.html yok"
                            fi

                            npm ci
                            npm test
                        '''
                    }

                    post {
                        always {
                            junit allowEmptyResults: true, testResults: 'jest-results/junit.xml'

                            publishHTML([
                                allowMissing: true,
                                alwaysLinkToLastBuild: false,
                                keepAll: false,
                                reportDir: 'playwright-report',
                                reportFiles: 'index.html',
                                reportName: 'Playwright HTML Report',
                                reportTitles: '',
                                useWrapperFileDirectly: true
                            ])
                        }
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.49.1-noble'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            ls -la

                            if [ ! -f build/index.html ]; then
                                echo "HATA: build/index.html bulunamadı. Önce npm run build çalışmalı."
                                exit 1
                            fi

                            npm install -g serve
                            serve -s build &
                            sleep 10

                            npx playwright test --reporter=html,junit
                        '''
                    }

                    post {
                        always {
                            junit allowEmptyResults: true, testResults: 'test-results/*.xml'

                            publishHTML([
                                allowMissing: true,
                                alwaysLinkToLastBuild: false,
                                keepAll: true,
                                reportDir: 'playwright-report',
                                reportFiles: 'index.html',
                                reportName: 'Playwright HTML Report',
                                reportTitles: '',
                                useWrapperFileDirectly: true
                            ])
                        }
                    }
                }
            }
        }
    }
}