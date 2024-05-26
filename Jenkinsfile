pipeline {
    agent any
    stages {
        stage('Inicio') {
            steps {
                echo "Esto código es de Jose Coca"
            }
        }
    stage('Get Code') {
            steps {
                git 'https://github.com/jcp1991/helloworld.git'
            }
        }
        stage('Print Workspace') {
            steps {
                echo "El espacio de trabajo es: ${WORKSPACE}"
                bat 'dir'
            }
        }
        stage('Build') {
            steps {
                echo "Ejecutando la etapa de Build..."
            }
        }
        
        stage('Paralelo') {
            parallel {
                stage('Unit') {
                    steps {
                        bat '''
                            set PYTHONPATH=%WORKSPACE%
                            pytest test\\unit
                            pytest test\\unit --junitxml=unit-results.xml
                        '''
                    }
                }
                stage('Service') {
                    steps {
                            bat '''
                                set FLASK_APP=app\\api.py
                                start flask run
                                start java -jar C:\\Users\\jose.coca\\Downloads\\instaladores\\wiremock-standalone-3.5.4.jar --port 9090 --root-dir test\\wiremock
                                set PYTHONPATH=.
                                pytest test\\rest
                                pytest test\\rest --junitxml=rest-results.xml
                            '''
                    }
                }
            }
        }
        
        stage('Publish JUnit Results') {
            steps {
                script {
                    junit 'unit-results.xml'
                    junit 'rest-results.xml'
                }
            }
        }
    }
}
