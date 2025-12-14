pipeline {
    agent any

    environment {
        GITHUB_TOKEN = credentials('github-token')
        VENV_DIR = ".venv"
        HOST = "0.0.0.0"
        PORT = "5000"
        APP_MODULE = "app:app"  
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/soukabadaoui88-sys/Jenkins-CI-CD.git',
                    credentialsId: 'github-token'
            }
        }

        stage('Setup Virtual Environment') {
            steps {
                // AJOUT DU CHEMIN PYTHON DANS LE PATH DE CE NOEUD PENDANT LE BUILD
                withEnv([
                    'PATH+PYTHON=C:\\Users\\souka\\AppData\\Local\\Programs\\Python\\Python313\\;C:\\Users\\souka\\AppData\\Local\\Programs\\Python\\Python313\\Scripts'
                ]) {
                    bat """
                    echo %PATH%  // Affiche le PATH pour verification
                    python -m venv ${VENV_DIR}
                    call ${VENV_DIR}\\Scripts\\activate
                    python -m pip install --upgrade pip
                    pip install -r requirements.txt
                    python --version
                    """
                }
            }
        }

        stage('Run Tests') {
            when {
                not {
                    changeset "README.md"
                }
            }
            parallel {
                stage('Test File 1') {
                    steps {
                        bat """
                        call ${VENV_DIR}\\Scripts\\activate
                        python -m pytest test_app.py -v
                        """
                    }
                }
            }
        }

        stage('Deploy (Local with Gunicorn)') {
            steps {
                echo 'Starting Flask app locally using Gunicorn...'
                bat """
                call ${VENV_DIR}\\Scripts\\activate
                start /B gunicorn --bind ${HOST}:${PORT} ${APP_MODULE} > gunicorn.log 2>&1
                echo ✅ Gunicorn started on http://${HOST}:${PORT}
                """
            }
        }
    }

    post {
        success {
            emailext(
                to: "EMAIL_TO_PLACEHOLDER",
                from: "EMAIL_FROM_PLACEHOLDER",
                replyTo: "EMAIL_REPLY_PLACEHOLDER",
                subject: "✅ SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                mimeType: 'text/html',
                body: """\
                <p>Good news!</p>
                <p>Build <b>${env.JOB_NAME} #${env.BUILD_NUMBER}</b> succeeded.</p>
                <p>Check details: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                """
            )
        }

        failure {
            emailext(
                to: "EMAIL_TO_PLACEHOLDER",
                from: "EMAIL_FROM_PLACEHOLDER",
                replyTo: "EMAIL_REPLY_PLACEHOLDER",
                subject: "❌ FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                mimeType: 'text/html',
                body: """\
                <p>Uh oh...</p>
                <p>Build <b>${env.JOB_NAME} #${env.BUILD_NUMBER}</b> failed.</p>
                <p>Check logs: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                """
            )
        }
    }
}
