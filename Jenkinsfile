pipeline {
    agent any

    environment {
        BANDIT_FILE = "bandit.json"
    }

    stages {
        stage('SAST-Bandit') {
            agent {
                docker {
                    image 'python:3.11-slim'
                    reuseNode true
                    args "-u root"
                }
            }
            steps {
                script {
                    sh '''
                        apt-get update -qq
                        apt-get install -qq -y git
                        git config --global --add safe.directory $(pwd)
                        pip install -q bandit
                    '''

                    try {
                        // Escaneo SAST con Bandit para PyGoat
                        sh "bandit -r . -f json -o ${BANDIT_FILE} || true"
                        sh "ls -lah ${WORKSPACE}"
                    } catch (err) {
                        unstable("Bandit encontró vulnerabilidades")
                    }
                }
            }
        }
    }

    post {

        success {
            echo "✅ SAST Bandit finalizó correctamente (sin errores de ejecución)"
        }

        failure {
            echo "❌ Falló el stage SAST Bandit (error de ejecución)"
        }

        always {
            echo "📦 Archivando reporte Bandit"
            archiveArtifacts artifacts: 'bandit.json', fingerprint: true, allowEmptyArchive: false
        }
    }
}
