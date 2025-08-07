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
                            # Garante que o diretório existe
                            mkdir -p test-results
                            
                            # Instala o reporter (se necessário)
                            npm install --save-dev jest-junit
                            
                            # Executa testes com configuração explícita
                            CI=true npm test -- --reporters=default --reporters=jest-junit
                            
                            # Debug: lista arquivos gerados
                            echo "Conteúdo de test-results:"
                            ls -la test-results/ || true
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
                            # Prepara ambiente
                            mkdir -p test-results
                            
                            # Instala dependências
                            npm install serve
                            
                            # Inicia servidor
                            node_modules/.bin/serve -s build &
                            SERVE_PID=$!
                            sleep 10
                            
                            # Executa testes com saída JUnit
                            npx playwright test --reporter=junit,html --output=test-results/junit-e2e.xml
                            
                            # Encerra servidor
                            kill $SERVE_PID
                            
                            # Debug: lista arquivos gerados
                            echo "Conteúdo do diretório:"
                            ls -la test-results/ || true
                            ls -la playwright-report/ || true
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
            sh 'echo "Artefatos arquivados:"; ls -la test-results/ playwright-report/ || true'
        }
    }
}