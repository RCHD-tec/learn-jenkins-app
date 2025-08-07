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
                    environment {
                        JEST_JUNIT_OUTPUT_DIR = 'test-results'
                        JEST_JUNIT_OUTPUT_NAME = 'junit-unit.xml'
                    }
                    steps {
                        sh '''
                            # Limpa node_modules para evitar erros
                            rm -rf node_modules/.cache
                            
                            # Garante que o diretório existe
                            mkdir -p test-results
                            
                            # Instala o reporter
                            npm install --save-dev jest-junit --no-audit --fund=false
                            
                            # Executa testes
                            CI=true npm test -- --reporters=default --reporters=jest-junit
                            
                            # Debug
                            echo "Conteúdo de test-results:"
                            ls -la test-results/
                        '''
                    }
                    post {
                        always {
                            junit 'test-results/junit-unit.xml'
                        }
                    }
                }

                stage('Testes E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.54.2-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            # Limpa node_modules para evitar erros
                            rm -rf node_modules/.cache
                            
                            # Prepara ambiente
                            mkdir -p test-results
                            
                            # Instala serve localmente (sem -g)
                            npm install serve --no-audit --fund=false
                            
                            # Inicia servidor em background
                            npx serve -s build &
                            SERVE_PID=$!
                            sleep 10
                            
                            # Executa testes
                            npx playwright test --reporter=junit,html --output=test-results/junit-e2e.xml || true
                            
                            # Encerra servidor
                            kill $SERVE_PID || true
                            
                            # Debug
                            echo "Conteúdo do diretório:"
                            ls -la test-results/ || true
                        '''
                    }
                    post {
                        always {
                            junit 'test-results/junit-e2e.xml'
                            publishHTML([
                                allowMissing: true,
                                alwaysLinkToLastBuild: true,
                                keepAll: true,
                                reportDir: 'playwright-report',
                                reportFiles: 'index.html',
                                reportName: 'Relatório Playwright'
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
            sh '''
                echo "Artefatos finais:"
                ls -la test-results/ playwright-report/ || true
            '''
        }
        success {
            echo 'Pipeline executado com sucesso!'
        }
        failure {
            echo 'Pipeline falhou. Verifique os logs para detalhes.'
        }
    }
}