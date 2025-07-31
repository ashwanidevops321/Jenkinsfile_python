pipeline {
    agent any

    environment {
        EXCLUDE_LIST = '.git .env __pycache__ venv'
        DEPLOY_SCRIPT = 'start_gunicorn.sh'
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Check for Changes') {
            steps {
                script {
                    def changes = sh(
                        script: "git diff --name-only HEAD~1 HEAD",
                        returnStdout: true
                    ).trim()

                    if (changes == '') {
                        echo "✅ No changes found. Skipping deployment."
                        currentBuild.result = 'SUCCESS'
                        error("No deployment needed.")
                    } else {
                        echo "📝 Changed files:\n${changes}"
                    }
                }
            }
        }

        stage('Deploy via Rsync') {
            steps {
                withCredentials([
                    string(credentialsId: 'deploy-user', variable: 'DEPLOY_USER'),
                    string(credentialsId: 'deploy-host', variable: 'DEPLOY_HOST'),
                    string(credentialsId: 'deploy-path', variable: 'DEPLOY_PATH')
                ]) {
                    sshagent (credentials: ['ssh-key-id-in-jenkins']) {
                        sh """
                            echo "🚀 Deploying changed files via rsync..."
                            rsync -avz --exclude=${EXCLUDE_LIST.replaceAll(' ', ' --exclude=')} ./ ${DEPLOY_USER}@${DEPLOY_HOST}:${DEPLOY_PATH}
                        """
                    }
                }
            }
        }

        stage('Run Remote Script') {
            steps {
                withCredentials([
                    string(credentialsId: 'deploy-user', variable: 'DEPLOY_USER'),
                    string(credentialsId: 'deploy-host', variable: 'DEPLOY_HOST'),
                    string(credentialsId: 'deploy-path', variable: 'DEPLOY_PATH')
                ]) {
                    sshagent (credentials: ['ssh-key-id-in-jenkins']) {
                        sh """
                            echo "🔧 Running deployment script on remote server..."
                            ssh ${DEPLOY_USER}@${DEPLOY_HOST} 'chmod +x ${DEPLOY_PATH}/${DEPLOY_SCRIPT} && bash ${DEPLOY_PATH}/${DEPLOY_SCRIPT}'
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment completed successfully!"
        }
        failure {
            echo "❌ Deployment failed. Check logs for details."
        }
    }
}
