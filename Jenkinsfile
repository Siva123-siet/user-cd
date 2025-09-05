pipeline {
    // Pre-build
    agent {
        label 'AGENT-1'
    }
    environment { 
        appVersion = ''
        REGION = "us-east-1"
        ACC_ID = "097003440739"
        PROJECT = "roboshop"
        COMPONENT = "user"
    }
    options {
        timeout(time: 30, unit: 'MINUTES') 
        disableConcurrentBuilds() 
    }
    parameters {
        string(name: 'appVersion', description: 'Image version of the application')
        choice(name: 'deploy_to', choices: ['dev', 'qa', 'prod'], description: 'Pick the Environment')
    }
    // Build 
    stages {
        stage('Deploy') {
            steps {
                script {
                  withAWS(credentials: 'aws-auth', region: 'us-east-1') {
                // Your AWS CLI commands or other AWS interactions go here
                sh """
                  aws eks update-kubeconfig --region $REGION --name "$PROJECT-${params.deploy_to}"
                  kubectl get nodes
                  kubectl apply -f 01-namespace.yaml
                  sed -i "s/IMAGE_VERSION/${params.appVersion}/g" values-${params.deploy_to}.yaml
                  helm upgrade --install $COMPONENT -f values-${params.deploy_to}.yaml -n $PROJECT .
               """
                }
                }
            }
        }
        stage('check status'){
            steps{
                script{
                withAWS(credentials: 'aws-auth', region: 'us-east-1'){
                def deploymentStatus = sh(script: 'kubectl rollout status deployment/user -n $PROJECT --timeout=30s || echo FAILED', returnStdout: true).trim()
                if (deploymentStatus.contains("successfully rolled out")) {
                     echo "Deployment is success"
                } else {
                   sh """
                      helm rollback $COMPONENT -n $PROJECT
                      sleep 20
                   """
                   def rollbackStatus = sh(script: 'kubectl rollout status deployment/user -n $PROJECT --timeout=30s || echo FAILED', returnStdout: true).trim()
                   if (rollbackStatus.contains("successfully rolled out")) {
                     error "Deployment is failure, Rollback Success"
                   }
                   else{
                     error "Deployment is failure, Rollback failure.Application is not running"
                   }
                }
            }
        }
    }
 }
 }      // API testing/Functional testing
        stage('Functional Testing'){
            when {
                    expression { params.deploy_to = "dev" }
                }
            steps{
                script{
                    echo "Run functional test cases"
                }
            }
        }
        // All components testing
        stage('Integration Testing'){
            when {
                    expression { params.deploy_to = "qa" }
                }
            steps{
                script{
                    echo "Run Integration test cases"
                }
            }
        }
        stage('Prod Deploy') {
            when {
                    expression { params.deploy_to = "prod" }
                }
            steps {
                script {
                  withAWS(credentials: 'aws-auth', region: 'us-east-1') {
                // Your AWS CLI commands or other AWS interactions go here
                sh """
                    echo "get cr number"
                    echo "check within the deployment window"
                    echo "is CR approved"
                    echo "trigger PROD deploy"
               """
                }
                }
            }
        }
    //post build
    post { 
        always { 
            echo 'I will always say Hello again!'
            deleteDir()
        }
        success { 
            echo 'Hello Success'
        }
        failure { 
            echo 'Hello failure'
        }
    }
}