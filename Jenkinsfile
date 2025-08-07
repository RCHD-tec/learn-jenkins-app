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

        stage('Testes') {
            parallel {
                stage('Testes Unitários') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            # Instala o reporter do JUnit
                            npm install --save-dev jest-junit
                            
                            # Roda testes com saída JUnit
                            CI=true npm test -- --reporters=default --reporters=jest-junit
                            
                            # Verifica se o relatório foi gerado
                            ls -la test-results || true
                        '''
                    }
                    post {
                        always {
                            junit 'test-results/junit.xml'
                        }
                    }
                }

                stage('Testes E2E') {
                    agent {
                        docker {
                            // Atualizado para a versão necessária
                            image 'mcr.microsoft.com/playwright:v1.54.2-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            # Instala o serve e roda a aplicação
                            npm install serve
                            node_modules/.bin/serve -s build &
                            SERVE_PID=$!
                            sleep 10
                            
                            # Roda testes do Playwright
                            npx playwright test --reporter=junit,html
                            
                            # Para o processo do serve
                            kill $SERVE_PID
                        '''
                    }
                    post {
                        always {
                            junit 'test-results/**/*.xml'
                            publishHTML([
                                allowMissing: true,
                                alwaysLinkToLastBuild: true,
                                keepAll: true,
                                reportDir: 'playwright-report',
                                reportFiles: 'index.html',
                                reportName: 'Relatório HTML Playwright'
                            ])
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'test-results/**/*, playwright-report/**/*'
        }
    }
}