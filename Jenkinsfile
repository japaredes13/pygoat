pipeline {
    agent any
    
    environment {
        VENV_DIR = 'venv'
        REPORTS_DIR = 'security-reports'
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo '=== Clonando Pygoat ==='
                git branch: 'master',
                    url: 'https://github.com/adeyosemanputra/pygoat.git'
            }
        }
        
        stage('Setup Python') {
            steps {
                echo '=== Configurando entorno Python ==='
                sh '''
                    python3 --version
                    python3 -m venv ${VENV_DIR}
                    . ${VENV_DIR}/bin/activate
                    pip install --upgrade pip
                    pip install bandit
                    bandit --version
                '''
            }
        }
        
        stage('SAST - Bandit') {
            steps {
                echo '=== Ejecutando análisis SAST ==='
                sh '''
                    mkdir -p ${REPORTS_DIR}
                    . ${VENV_DIR}/bin/activate
                    
                    # Análisis completo
                    bandit -r . -ll -f json -o ${REPORTS_DIR}/bandit-report.json || true
                    bandit -r . -ll -f html -o ${REPORTS_DIR}/bandit-report.html || true
                    bandit -r . -ll -f txt -o ${REPORTS_DIR}/bandit-report.txt || true
                    
                    # Mostrar en consola
                    echo "=== RESUMEN DE VULNERABILIDADES ==="
                    cat ${REPORTS_DIR}/bandit-report.txt
                '''
            }
        }
        
        stage('Archive Reports') {
            steps {
                echo '=== Archivando reportes ==='
                
                archiveArtifacts artifacts: "${REPORTS_DIR}/**/*",
                                 fingerprint: true
                
                publishHTML(target: [
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: "${REPORTS_DIR}",
                    reportFiles: 'bandit-report.html',
                    reportName: 'Bandit SAST Report'
                ])
            }
        }
    }
    
    post {
        always {
            echo '=== Limpieza ==='
            sh "rm -rf ${VENV_DIR}"
        }
        success {
            echo '✅ Pipeline completado'
        }
        failure {
            echo '❌ Pipeline falló'
        }
    }
}