pipeline {
    agent any

    environment {
        DOCKER_HUB_LOGIN = credentials('docker-hub')
        VERSION = sh(script: 'jq --raw-output .version package.json', returnStdout: true).trim()
        REPO = sh(script: 'basename `git rev-parse --show-toplevel`', returnStdout: true).trim()
        REGISTRY = credentials('registry-hub')
        SNYK_CREDENTIALS = credentials('snyk-token')
        SCANNER_HOME=tool 'sonar-scanner'

    }

    stages {
               
        stage('Install Dependencies') {
            agent {
                docker {
                    image 'node:16-alpine'
                    args '-u root:root'
                }
            }
            steps {
                script {
                    sh 'npm install'
                }
            }
        }  //stage Install Dependencies
    
        
        stage ('Security SAST') {
          parallel {
             stage('Gitleaks-Scan') {
                    agent {
                        docker {
                            image 'zricethezav/gitleaks'
                            args '--entrypoint="" -u root -v ${WORKSPACE}:/src'
                        }
                    }                    
                    steps {
                        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                            script {
                                sh "gitleaks detect --verbose --source . -f json -r  report_gitleaks.json"
                                stash includes: 'report_gitleaks.json', name: 'report_gitleaks.json'
                            }
                        }
                    }
                }
               stage('Semgrep-Scan') {
                    agent {
                        docker {
                            image 'returntocorp/semgrep'
                            args '-u root:root -v ${WORKSPACE}:/src'
                        }
                    }                     
                    steps {
                        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                            script {
                                sh "semgrep ci --json --exclude=package-lock.json --output report_semgrep.json --config auto --config p/ci"
                                stash includes: 'report_semgrep.json', name: 'report_semgrep.json'
                            }
                        }
                    }
                }    
               stage('Snyk Test') {
                   agent {
                   docker {
                   image 'snyk/snyk:node'
                    args '--entrypoint="" -e SNYK_TOKEN=$SNYK_CREDENTIALS -u root:root -v ${WORKSPACE}:/src'
                   }
                  }     
                   steps {
                     catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                     script {
                     sh "snyk test --json --file=package.json --severity-threshold=high --print-deps --print-deps-uses --print-vulnerabilities --print-trace --print-all-environment --json-file-output=report_snyk.json"
                     stash includes: 'report_snyk.json', name: 'report_snyk.json'
                      }
                    }
                   }
                }

               stage('NPMAudit-Scan') {
                    agent {
                        docker {
                            image 'node:16-alpine'
                            args '-u root:root -v ${WORKSPACE}:/src'
                        }
                    }                    
                    steps {
                        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                            script {
                                sh "npm audit --registry=https://registry.npmjs.org -audit-level=moderate --json > report_npmaudit.json"
                                stash includes: 'report_npmaudit.json', name: 'report_npmaudit.json'
                            }
                        }
                    }
                }

                stage("Sonarqube Analysis "){
                steps{
                    withSonarQubeEnv('sonar-scanner') {
                        sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=CICD \
                        -Dsonar.projectKey=token-sonar-nuevo '''
                    }
                }
                }

            } //parallel
        
        } //stage SAST

    

         stage('Docker Build') {
            steps {
                script {
                    sh "docker build -t $REGISTRY/$REPO:$VERSION ."
                }
            }
        }

         //stage Build
         
         stage('Trivy-Scan') {
            agent {
                docker {
                    image 'aquasec/trivy:0.48.1'
                    args '--entrypoint="" -u root -v /var/run/docker.sock:/var/run/docker.sock -v ${WORKSPACE}:/src'
                }
            }
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    
                    script {
                        sh "trivy image --timeout 10m --format json --ignore-unfixed  --debug  -o report_trivy.json $REGISTRY/$REPO:$VERSION" 
                        stash includes: 'report_trivy.json', name: 'report_trivy.json'
                    }
                }
            }
        } //stage Trivy

        

        stage('Docker Push') {
            steps {
               script {
                    sh '''
                    docker login -u $DOCKER_HUB_LOGIN_USR -p $DOCKER_HUB_LOGIN_PSW
                    docker push $REGISTRY/$REPO:$VERSION
                    '''      
                }
            }
         } //stage push

        
        stage('Deploy to Kubernetes') {
            steps {
               // echo 'Desplegando la aplicación to Kubernetes'
               sshagent (['ssh-agent']){
                sh 'ssh -tt -o StrictHostKeyChecking=no vagrant@192.168.0.92 kubectl get nodes'
                sh 'scp -o StrictHostKeyChecking=no "./deployment.yaml" "vagrant@192.168.0.92:/vagrant/game2048/"'
                sh 'ssh -tt -o StrictHostKeyChecking=no vagrant@192.168.0.92 kubectl apply -f /vagrant/game2048/deployment.yaml'
                }
            }
        }

        stage('Security DAST') {
            agent {
                  docker {
                  image 'zaproxy/zap-stable'
                  args '-u root:root -v ${WORKSPACE}:/zap/wrk:rw'
               }
               }
            
            steps {
              //  echo 'Testing en tiempo real OWASP ZAP..'
  
             catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
              script {
                // Run OWASP ZAP with baseline scan against your target URL
                sh """
                zap-baseline.py -t http://192.168.0.92:3001 -g gen.conf -r zap_report.html || true
                """
                // Stash the generated reports
                stash includes: 'zap_report.html', name: 'zap_reports.html'
             } 
            } 
           } 
         }
         
        

       } //all stages
}

