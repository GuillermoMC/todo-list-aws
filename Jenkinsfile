pipeline {
    
    agent any
    
    environment {
        
        GITHUB_TOKEN = 'unic-todo-token-access'
        
    }
    
    stages {
        
        stage('Get Code') {

            steps {

                script {

                    def gitCredentialsId = GITHUB_TOKEN
                    checkout([$class: 'GitSCM', branches: [[name: '*/master'], [name: '*/develop']], userRemoteConfigs: [[url: 'https://github.com/GuillermoMC/todo-list-aws.git', credentialsId: gitCredentialsId]]])
                
                }
                
                sh '''

                    git checkout develop

                '''

            }
        }
        
      
        stage('Static Test') {
            
            parallel  {
                
                stage('Static') {
            
                    steps {
        
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
        
                            sh '''

                                flake8 --exit-zero --format=pylint src > flake8.out

                            '''
        
                            recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates: [[threshold: 0, type: 'TOTAL', unstable: true], [threshold: 1500, type: 'TOTAL', unstable: false]]
        
                        }
                    
                    }
                    
                }
                
                stage('Security') {
                    
                    steps {
        
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            
                            sh '''

                                bandit --exit-zero -r . -f custom -o bandit.out --msg-template "{abspath}:{line}: [{test_id}] {msg}"

                            '''
        
                            recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')], qualityGates: [[threshold: 0, type: 'TOTAL', unstable: true], [threshold: 1500, type: 'TOTAL', unstable: false]]
                            
                        }
        
                    }
                    
                }
                        
                
            }
            
            
        }
        
        stage('Deploy') {
            
            steps {

                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                
                    sh '''

                        sam build
                    
                    '''

                    // tener en cuenta de que tiene que estar el s3 creado
                    // --no-execute-changeset se queda en CREATE_IN_PROGRESS
                    sh '''

                        sam deploy --no-fail-on-empty-changeset --config-file samconfig.toml --config-env staging --s3-bucket unir-cp-sam --stack-name todo-aws-list-staging
                    
                    '''

                }
                
            }
            
        }
        
        stage('Rest Test') {
            
            steps {
                
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {

                    sh '''
                    
                        variable=$(aws cloudformation describe-stacks --stack-name todo-aws-list-staging --query "Stacks[0].Outputs[?OutputKey=='BaseUrlApi'].OutputValue" --output text)
                        
                        export BASE_URL=$variable
                        
                        pytest --junitxml=result-unit.xml test/integration/todoApiTest.py
                        
                    '''
                    
                    junit 'result*.xml'
                    
                }
                
            }
            
        }
        
        stage('Promote') {
            
            steps {
                
               script {
                   
                   withCredentials([usernamePassword(credentialsId: GITHUB_TOKEN, usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_TOKEN')]) {
                       
                        // echo "creando fichero" > fichero_subida.txt
                        // git add fichero_subida.txt
                        // git commit -m "Subiendo un fichero nuevo desde un pipeline de Jenkins en AWS"
                        // git push https://\$GIT_USERNAME:\$GIT_TOKEN@github.com/GuillermoMC/todo-list-aws.git develop

                        sh '''

                            git checkout master
                            git merge --no-ff develop
                            git push https://\$GIT_USERNAME:\$GIT_TOKEN@github.com/GuillermoMC/todo-list-aws.git master 

                        '''
                        
                    }
                   
               }
                
            }
            
        }
        
    }
    
    post {
        
        always {
            
            cleanWs()

        }

    }
    
}
