pipeline {
    agent any
    stages {
        stage('Inicio') {
            steps {
                echo "Este código es de Jose Coca"
            }
        }
        stage('Get Code') {
            steps {
                git 'https://github.com/jcp1991/caso-b.git'
            }
        }
        stage('Print Workspace') {
            steps {
                echo "El espacio de trabajo es: ${WORKSPACE}"
                bat 'dir'
            }
        }
        stage('Static') {
            steps {
                bat '''
                    flake8 --exit-zero --format=pylint app >flake8.out
					'''
					recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates : [[threshold: 10, type: 'TOTAL', unstable: true], [threshold: 11, type: 'TOTAL', unstable: false]]
            }
        }
        stage('Seguridad') {
            steps {
                bat '''
                    bandit --exit-zero -r . -f custom -o bandit.out --severity-level medium --msg-template "{abspath}:{line}: [{test_id}]{msg}" 
                '''
					recordIssues tools: [bandit(pattern: 'bandit.out')],  qualityGates: [[threshold: 2, type: 'TOTAL', unstable: true],  [threshold: 4, type: 'TOTAL', unstable: false]]
            }
        }
        stage('Cobertura (Coverage)') {
            steps {
                bat '''
                    set PYTHONPATH=%WORKSPACE%
                    coverage run -m pytest
                    coverage report
                    coverage html
                '''
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
    }
}
