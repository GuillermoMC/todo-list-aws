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
                    
                    checkout([$class: 'GitSCM', branches: [[name: '*/master']], userRemoteConfigs: [[url: 'https://github.com/GuillermoMC/todo-list-aws.git', credentialsId: gitCredentialsId]]])
                
                    
                }
                
                sh '''

                    git checkout master
                    
                '''

            }
        }
        
        stage('Deploy') {
            
            steps {
                
                sh '''
                
                    sam build
                
                '''
                
                // tener en cuenta de que tiene que estar el s3 creado
                // --no-execute-changeset se queda en CREATE_IN_PROGRESS
                sh '''
                
                    sam deploy --no-fail-on-empty-changeset --config-file samconfig.toml --config-env production --s3-bucket unir-cp-sam --stack-name todo-aws-list-production
                
                '''
                
            }
            
        }
        
        stage('Rest Test') {
            
            steps {
                
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {

                    sh '''
                    
                        variable=$(aws cloudformation describe-stacks --stack-name todo-aws-list-production --query "Stacks[0].Outputs[?OutputKey=='BaseUrlApi'].OutputValue" --output text)
                        
                        export BASE_URL=$variable
                        
                        pytest --junitxml=result-unit.xml test/integration/todoApiTest.py
                        
                    '''
                    
                    junit 'result*.xml'
                    
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