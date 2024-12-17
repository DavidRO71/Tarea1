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
                git 'https://github.com/DavidRO71/Tarea1.git'
                echo 'GetCode'
            }
        }

        stage('Build') {
            steps {
                bat "%PYTHON_HOME%\\python --version"
                echo 'No hay que compilar nada es Python'
            }
        }

   
        stage('ExecCalc') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    bat "%PYTHON_HOME%\\python app\\calc.py"  
                }
            }
        }

        stage('Tests'){
            parallel {
                stage('TestUnit') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            bat "%PYTHON_HOME%\\python -m pip install pytest"
                            bat "%SCRIPT_HOME%\\pytest --junitxml=result-unit.xml test\\unit"
                        }
                    }
                }
        
                stage('TestRest') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            bat "%PYTHON_HOME%\\python -m pip install flask"
                            bat "set flask_app=app\\api.py"
                            bat "start %SCRIPT_HOME%\\flask run"
                            bat "start java -jar C:\\WorKir\\UNIR\\MODULO_2\\Wiremock\\wiremock-standalone-3.10.0.jar --port 9090 --root-dir C:\\ProgramData\\Jenkins\\.jenkins\\workspace\\O24\\Tarea1\\test\\wiremock\\mappings"
                            bat "set PYTHONPATH = '.'"
                            //Esperamos x segundos para que el servicio se arranque del todo. En este caso he puesto 5 segundos
                            sleep 5
                            bat "%SCRIPT_HOME%\\pytest --junitxml=result-rest.xml test\\rest"
                            stash includes: '**', name: 'testRest'
                        }
                    }
                }
            }
        }

        stage('Grafica') {
            steps {
                junit 'result*.xml'
            }
        }
    }
}

