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
                //Almacenamos el codigo en un almacen temporal para luego poder tener acceso a el en otro agente
                stash includes: '**', name: 'codigo'
            }
        }

        stage('Build') {
            steps {
                bat "%PYTHON_HOME%\\python --version"
                echo 'No hay que compilar nada es Python'
            }
        }

   
        stage('ExecCalc') {
            agent { 
                //Llamamos al agente1
                label 'agente1' 
            }
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    //Mostramos el nombre del agente
                    echo "NODE_NAME = ${env.NODE_NAME}"
                    //Mostramos el workspace
                    echo "WORKSPACE = ${WORKSPACE}"

                    //Recuperamos el codigo que hemos almacenado anteriormente
                    unstash 'codigo'
                    bat "%PYTHON_HOME%\\python app\\calc.py"  
                }
            }
        }

        stage('Tests'){
            parallel {
                stage('TestUnit') {
                    agent { 
                        //Llamamos al Agente2
                        label 'Agente2' 
                    }
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            //Mostramos el nombre del agente
                            echo "NODE_NAME = ${env.NODE_NAME}"
                            //Mostramos el workspace
                            echo "WORKSPACE = ${WORKSPACE}"
                            
                            //Recuperamos el codigo que hemos almacenado anteriormente
                            unstash 'codigo'
                            bat "%PYTHON_HOME%\\python -m pip install pytest"
                            bat "%SCRIPT_HOME%\\pytest --junitxml=result-unit.xml test\\unit"
                            stash includes: '**', name: 'testUni'
                        }
                    }
                }
        
                stage('TestRest') {
                    agent { 
                        //Llamamos al Agente3
                        label 'Agente3' 
                    }
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            //Mostramos el nombre del agente
                            echo "NODE_NAME = ${env.NODE_NAME}"
                            //Mostramos el workspace
                            echo "WORKSPACE = ${WORKSPACE}"
                            
                            //Recuperamos el codigo que hemos almacenado anteriormente
                            unstash 'codigo'
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
                unstash 'testUni'
                junit 'result-unit.xml'
                unstash 'testRest'
                junit 'result-rest.xml'
            }
        }
    }
}

