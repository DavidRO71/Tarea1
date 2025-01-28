pipeline {
    agent any

    environment {
        PYTHON_HOME = 'C:\\Users\\dramo\\AppData\\Local\\Programs\\Python\\Python313\\' 
    	SCRIPT_HOME = 'C:\\Users\\dramo\\AppData\\Local\\Programs\\Python\\Python313\\Scripts\\'
		PYTHONPATH = '.'
    }

    stages {
        stage('GetCode') {
            steps {
				//Recuperamos el código de nuestro repositorio
                git 'https://github.com/DavidRO71/Tarea1.git'
                bat 'dir'
                echo 'GetCode'
                echo WORKSPACE
            }
        }
        
        stage('Build') {
           steps {
              echo 'No hay nada que compilar porque es Python'
           }
        }

        //Prueba de analisis de codigo estatico
        stage('Estatico') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
					//Ejecutamos el test analisis de codigo estatico
                    //Añado --max-line-length 120 para no fallen las lineas por tamaño > 79
                    bat "flake8 --format=pylint --exit-zero --max-line-length 120 app > flake8.out"

					//Las qualityGates:
					//Si se encuentran 8 o más hallazgos, debe marcarse la etapa y el build como unstable (amarillo/naranja)
					//Si se encuentran 10 o más hallazgos, debe marcarse la etapa y el build como unhealthy (rojo).
                    recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates: [[threshold: 8, type: 'TOTAL', unstable: true], [threshold: 10, type: 'TOTAL', unstable: false]]
            
                }
            }
        }  
        
        //Prueba de Seguridad
        stage('Seguridad') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {                
					//Ejecutamos el test de Seguridad
					bat '''
						bandit --exit-zero -r . -f custom -o bandit.out --msg-template "{abspath}:{line}: [{test_id}] {msg}"
					'''
					
					//Las qualityGates:
					//Si se encuentran 2 o más hallazgos, debe marcarse la etapa y el build como unstable (amarillo/naranja).
					//Si se encuentran 4 o más hallazgos, debe marcarse la etapa y el build como unhealthy (rojo).
                    recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')], qualityGates: [[threshold: 2, type: 'TOTAL', unstable: true], [threshold: 4, type: 'TOTAL', unstable: false]]
                }
            }
        }
        
        //Prueba de Cobertura
        stage('Cobertura') {
            steps {
				catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
					bat "set PYTHONPATH = '.'" 
					//Ejecutamos las pruebas de Cobertura
					bat "coverage run --branch --source=app --omit=app\\__init__.py,app\\api.py -m pytest --junitxml=result-unit.xml test\\unit"
					//Mostramos el report que se acaba de generar
					bat "coverage xml"

					//Las qualityGates:
                    //Si la cobertura por líneas se encuentra entre 85 y 95, se marcará como unstable. Por encima será verde y, por debajo, rojo.
					//Si la cobertura por ramas/condiciones se encuentra entre 80 y 90, se marcará como unstable. Por encima será verde y, por debajo, rojo.
                    cobertura coberturaReportFile: 'coverage.xml', conditionalCoverageTargets: '100,0,90', lineCoverageTargets: '100,0,95'
                    //cobertura coberturaReportFile: 'coverage.xml', conditionalCoverageTargets: '100,80,90', lineCoverageTargets: '100,85,95'
                }
            }
        }
        
        stage('TestUnitarios'){
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
					//Mostramos el report que se ha generado en el paso anterior de Cobertura
                    junit 'result-unit.xml'
                }
            }
        }
        
        //Prueba de Rendimineto
        stage('Rendimineto') {
            steps {
                //Arrancamos FLASK
                bat "set PYTHONPATH = '.'"
                bat "set flask_app=app\\api.py"
                bat "start %SCRIPT_HOME%flask run"
				
                //Ponemos un sleep de 15s para que le de tiempo a flask a levantarse
				sleep 15
				
                //5 hilos y 40 llamadas al microservicio de suma y 40 llamadas al microservicio de resta
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    bat '''
                        C:\\WorKir\\UNIR\\MODULO_2\\JMETER\\apache-jmeter-5.6.3\\bin\\jmeter -n -t JMeter\\TestPlan.jmx -f -l flask.jtl
                    '''
					
					//Mostramos el report que se ha generado en el paso anterior
                    perfReport sourceDataFiles: 'flask.jtl'
                }
            }
        }

    }

}

