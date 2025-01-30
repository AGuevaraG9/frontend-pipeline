pipeline {
    agent any
    environment {
        VERCEL_TOKEN = credentials('VERCEL_TOKEN')
    }

    stages {
        stage ('Instalar dependencias...') {
            agent {
                docker { image 'node:22-alpine'}
            }
            steps {
                sh 'npm install'
            }
        }

        stage ('Construir proyecto con archivos estaticos...') {
            agent {
                docker { image 'node:22-alpine'}
            }
            steps {
                sh 'npm run build'
            }
        }

        stage ('Deploy hacia Vercel...') {
            agent {
                docker { image 'node:22-alpine'}
            }
            steps {
                sh """
                    npm install -g vercel
                    vercel deploy --name frontend-pipeline --yes --prod --token $VERCEL_TOKEN
                """
            }
        }
    }
}