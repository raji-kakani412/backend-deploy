pipeline {
    agent {
        label 'AGENT-1'
    }
    options{
        timeout(time:30, unit:'MINUTES')
        disableConcurrentBuilds()
        //retry(1)
    }
    parameters{
        //booleanParam(name: 'deploy', defaultValue: false, description: 'Select to deploy or not')
        choice(name: 'ENVIRONMENT', choices:['dev','qa','uat','pre-prod','prod'], description: 'select your environment')
        string(name: 'version', description:"Enter your app version")
        string(name: 'jira-id', description:"Enter your jira id")
    }
    environment {
        appVersion= ''
        region='us-east-1'
        account_id= ''
        project= 'expense'
        environment=''
        component= 'backend'
    }

    stages {
        stage('Setup Environment'){
            steps{
                script{
                    environment= params.ENVIRONMENT
                    appVersion= params.version
                    accountID = pipelineGlobals.getAccountID(environment)
                }
            }
        }
        stage('Integration Tests'){
            when{
                expression{ params.ENVIRONMENT == 'qa'}
            }
            steps{
                script{
                   sh """
                        echo "Run Integration Tests"
                   """
                }
            }
        }
        stage('Check JIRA'){
             when{
                expression{ params.ENVIRONMENT == 'prod'}
            }
            steps{
                script{
                   sh """
                        echo "check jira status"
                        echo "Check deployment window"
                        echo "Fail pipeline if above two are not true "
                   """
                }
            }
        }
        stage('Deploy'){
            steps{
                withAWS(region: 'us-east-1', credentials: 'aws-creds') {
                    sh """
                        aws eks update-kubeconfig --region ${region} --name ${project}-${environment}
                        cd helm
                        sed -i 's/IMAGE_VERSION/${appVersion}/g' values-${environment}.yaml
                        helm upgrade --install ${component} -n ${project} -f values-${environment}.yaml .
                    """
                }    
            }
        }
        
    }
    post{
        always{
            sh 'echo This runs always'
            deleteDir()
        }
        success{
            sh 'echo This runs when pipeline is success'
        }
        failure{
            sh 'echo This runs when pipeline fails'
        }
    }
}