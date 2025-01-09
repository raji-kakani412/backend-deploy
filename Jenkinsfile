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
        booleanParam(name: 'deploy', defaultValue: false, description: 'Select to deploy or not')
        choice(name: 'ENVIRONMENT', choices:['dev','qa','uat','pre-prod','prod'], description: 'select your environment')
        string(name: 'version', description:"Enter your app version")
    }
    environment {
        DEBUG = 'true'
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
                }
            }
        }
        stage('Deploy'){
            when{
                expression {params.deploy}
            }
           steps{
                withAWS(region: 'us-east-1', credentials: 'aws-creds') {
                    sh """
                        aws eks update-kubeconfig --region ${region} --name ${project}-${environment}
                        cd helm
                        sed -i 's/IMAGE_VERSION/${appVersion}/g' values-${environement}.yaml
                        helm upgrade --install ${component} -n ${project} -f values-${environement}.yaml
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