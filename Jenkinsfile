pipeline {
    agent any
    stages {
        stage('Get Code') {
            steps {
				git branch: 'feature_fix_coverage', url: 'https://github.com/jcp1991/caso-b.git'
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
					catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE'){
                bat '''
                    flake8 --exit-zero --format=pylint app >flake8.out
					'''
					recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates : [[threshold: 8, type: 'TOTAL', unstable: true], [threshold: 10, type: 'TOTAL', unhealthy: true]]}
            }
        }
        stage('Security') {
            steps {
					catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                bat '''
                    bandit --exit-zero -r . -f custom -o bandit.out --severity-level medium --msg-template "{abspath}:{line}: [{test_id}] {msg}"
                '''
					 recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')],  qualityGates: [[threshold: 2, type: 'TOTAL', unstable: true],  [threshold: 4, type: 'TOTAL', unstable: false]]}
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
                            timeout /T 12
							set PYTHONPATH=.
                            pytest test\\rest
                            pytest test\\rest --junitxml=rest-results.xml
                        '''
                    }
                }
            }
        }
				stage('Cobertura (Coverage)') {
					steps {
						bat '''
							coverage combine
							coverage report
							coverage xml -o coverage.xml
					'''
					catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
						cobertura coberturaReportFile: 'coverage.xml', conditionalCoverageTargets: '100,100,100', lineCoverageTargets: '100,100,100', onlyStable: false
                }
            }
        }
		
		stage('Jmeter (Perfomance)') {
            steps {
                bat 'C:\\Users\\jose.coca\\Downloads\\instaladores\\apache-jmeter-5.6.3\\apache-jmeter-5.6.3\\bin\\jmeter -n -t test\\jmeter\\flask.jmx -f -l flask.jtl'
				perfReport sourceDataFiles : 'flask.jtl'
            }
        }
	
	}
}

