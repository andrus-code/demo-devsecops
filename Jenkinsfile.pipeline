pipeline {
    agent any

       //environment {
       //  DOCKER_HUB_LOGIN = credentials('docker-hub')
       //  DOCKER_ARGS = '-u root:root -v $HOME/.npm:/.npm'
       //  VERSION = sh(script: 'jq --raw-output .version package.json', returnStdout: true).trim()
       //  REPO = sh(script: 'basename `git rev-parse --show-toplevel`', returnStdout: true).trim()
       //  REGISTRY = credentials('registry-hub')
       //  SNYK_CREDENTIALS = credentials('snyk-token')
    //}

    stages {

        stage ('Init') {
         parallel {
            stage('Install Dependencies') {
                steps {
                    echo "Prueba de dependencias"
                }
            }
            stage ('Unit tests') {
                steps {
                    echo "Test de dependencias"
             }
                 
           }
         }
        }

        stage('Security SAST') {
            parallel {
                stage('Horusec') {
                                      
                    steps {
                        echo "prueba Horusec"
                    }
                }

                stage('NPMAudit-Scan') {
                    steps {
                        echo 'prueba NPMAudit'
                    }
                }
                stage('Semgrep-Scan') {
                    steps {
                        echo 'prueba Semgrep'
                    }
                }    
            } // end paralles
        } //end SAST
        
        stage('Docker Build') {
            steps {
                echo "Prueba de build "
                }
        }

        stage('Container Security Scan') {
            parallel {
                stage ('Trivy Scan') {
                    steps {
                    echo "Prueba Trivy"
                    }
                }
                  stage ('Linter Scan') {
                    steps {
                    echo "Prueba Linter"
                    }
                }
             }
        } //end SCAN

        stage('Update & Push') {
            parallel {
                stage ('Docker Push') {
                    steps {
                    echo "Prueba docker push"
                    }
                }
                  stage ('update Compose') {
                    steps {
                    echo "Prueba update compose"
                    }
                }
             }
        } // end update

        stage('Security DAST') {
            steps {
                echo "Prueba de DSAT"
                }
        }

        stage('Deploy') {
            steps {
                echo "Prueba de deploy"
                }
        }
        

    } // end stages
} // end pipeline
