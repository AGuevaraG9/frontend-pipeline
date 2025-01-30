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

        stage('Validar conexion AWS ...') {
            agent {
                docker { 
                    image 'amazon/aws-cli:2.23.7'
                    args '--entrypoint ""'
                }
            }
            steps {
                withAWS(credentials: 'aws-credentials-s3', region: 'us-east-1') {
                    script {
                        def buckets  = sh(returnStdout: true, script: 'aws s3 ls').trim()
                        echo "Buckets disponibles en aws: \n${buckets}"
                        
                        def folders = sh(returnStdout: true, script: 'aws s3 ls s3://bucket-codigo-backup/AbelGuevara/').trim()
                        echo "Carpetas disponibles en el bucket 'bucket-codigo-backup/AbelGuevara': \n${folders}"

                        def vercelBackup = sh(returnStdout: true, script: 'aws s3 ls s3://bucket-codigo-backup/AbelGuevara/vercel/').trim()
                        echo "Versiones disponibles para vercel': \n${vercelBackup}"
                    }
                }
            }
        }

        stage('Subir archivos al bucket de respaldo ...') {
            agent {
                docker {
                    image 'amazon/aws-cli:2.23.7'
                    args '--entrypoint ""'
                }
            }
            steps {
                withAWS(credentials: 'aws-credentials-s3', region: 'us-east-1') {
                    script {
                        def ultimaCarpetaDeBackup = sh(returnStdout: true, script: '''
                            aws s3 ls s3://bucket-codigo-backup/AbelGuevara/vercel/ | awk '{print $2}' | grep VERSION_ | sort | tail -n 1
                        ''').trim()

                        echo "Ultima carpeta del bucket backup: ${ultimaCarpetaDeBackup}"

                        def baseVersion = 'VERSION_1.0'

                        if (ultimaCarpetaDeBackup) {
                            def currentVersion = ultimaCarpetaDeBackup.replace('VERSION_','').replace('/', '')

                            echo "Version actual: ${currentVersion}"

                            def versionNumber = currentVersion.toFloat() + 0.1

                            echo "Numero de Version aumentado : ${versionNumber}"

                            baseVersion = String.format("VERSION_%.1f", versionNumber)

                            echo "Nombre de version formateado : ${baseVersion}"
                        }

                        echo "Subiendo los archivos al bucket s3 en la carpeta ${baseVersion}..."
                        sh """
                            aws s3 sync dist/ s3://bucket-codigo-backup/AbelGuevara/vercel/${baseVersion}/ --delete
                        """
                    }                   
                }
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