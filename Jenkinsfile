pipeline {
    agent any
    stages {
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
        stage('Security') {
            steps {
                bat '''
                    bandit --exit-zero -r . -f json -o bandit.out --severity-level medium
                '''
					catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') { recordIssues tools: [bandit(pattern: 'bandit.out')],  qualityGates: [[threshold: 2, type: 'TOTAL', unstable: true],  [threshold: 4, type: 'TOTAL', unstable: false]]}
            }
        }
        stage('Cobertura (Coverage)') {
            steps {
                bat '''
                    coverage run --source=app --omit=app\\\\__init__.py;app\\apy.py -m pytest test\\unit
					'''
                    catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {cobertura coberturaReportFile: 'coverage.xml', conditionalCoverageTargets: '100,90,80', lineCoverageTargets: '100,95,85', onlyStable: false}
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

