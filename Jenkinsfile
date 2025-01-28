pipeline {
    agent any

    stages {
        stage ('Instalar dependencias...') {
            agent {
                docker { image 'node:22-alpine'}
            }
            steps {
                sh 'npm cache clean --force'
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

        stage('Validar imagen de AWS ...') {
            agent {
                docker { 
                    image 'amazon/aws-cli:2.23.7'
                    args '--entrypoint ""'
                }
            }
            steps {
                echo "Usando aws CLI.."
                sh 'aws --version'
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
                    }
                }
            }
        }

        stage('Subir proyecto al bucket s3 AWS ...') {
            agent {
                docker {
                    image 'amazon/aws-cli:2.23.7'
                    args '--entrypoint ""'
                }
            }
            steps {
                withAWS(credentials: 'aws-credentials-s3', region: 'us-east-1') {
                    script {
                        def timestamp = new Date().format('yyyy_MM_dd_HH_mm_ss')
                        echo "Creando backup con fecha ${timestamp}"
                        sh "aws s3 sync dist/ s3://bucket-codigo-backup/AbelGuevara/${timestamp}/"

                        echo "Subiendo los archivos al bucket s3..."
                        sh '''
                            aws s3 sync dist/ s3://bucket-codigo-abel --delete
                        '''
                    }                   
                }
            }
        }
    }
}