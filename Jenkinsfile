stage('Testes E2E') {
    agent {
        docker {
            image 'mcr.microsoft.com/playwright:v1.54.2-jammy'
            reuseNode true
        }
    }
    steps {
        sh '''
            # Limpa cache e prepara ambiente
            rm -rf node_modules/.cache test-results playwright-report
            mkdir -p test-results
            
            # Instala serve localmente
            npm install serve --no-audit --fund=false
            
            # Inicia servidor
            npx serve -s build -l 5000 &
            SERVE_PID=$!
            sleep 10
            
            # Executa testes com saída JUnit em localização explícita
            npx playwright test --reporter=junit,html --output=test-results/e2e-junit.xml
            
            # Encerra servidor
            kill $SERVE_PID || true
            
            # Move relatório para localização padrão se necessário
            [ -f test-results/junit-e2e.xml/test-results.xml ] && \
              mv test-results/junit-e2e.xml/test-results.xml test-results/e2e-junit.xml
            
            # Debug
            echo "Estrutura de arquivos:"
            find test-results/ -type f
        '''
    }
    post {
        always {
            // Padrão mais abrangente para capturar relatórios
            junit 'test-results/**/*.xml'
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