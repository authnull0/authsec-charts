pipeline {
    agent any

    environment {
        AZURE_SUBSCRIPTION_ID = credentials('subscriptionId')
        AKS_CLUSTER = 'authnull-v2'  
        RESOURCE_GROUP = 'azure-k8s'
        K8S_NAMESPACE = 'authsec'
        AZURE_CLIENT_ID = credentials('clientid')  
        AZURE_CLIENT_SECRET = credentials('secretid') 
        AZURE_TENANT_ID = credentials('tenantid')  
        GITHUB_REPO = 'https://github.com/authnull0/authsec-charts.git'
        GITHUB_BRANCH = 'main'
    }

    triggers {
        githubPush()
    }

    options {
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '10', daysToKeepStr: '', numToKeepStr: '10')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${GITHUB_BRANCH}", credentialsId: 'pshussain-github', url: "${GITHUB_REPO}"
            }
        }

        stage('Authenticate to AKS') {
            steps {
                script {
                    sh '''
                       az login --service-principal -u "$AZURE_CLIENT_ID" -p "$AZURE_CLIENT_SECRET" --tenant "$AZURE_TENANT_ID"
                       az account set --subscription "$AZURE_SUBSCRIPTION_ID"
                       az aks get-credentials --resource-group "$RESOURCE_GROUP" --name "$AKS_CLUSTER" --admin --overwrite-existing
                    '''

                }
            }
        }

        stage('Deploying Helm Chart') {
            steps {
                sh "helm upgrade dev . --namespace ${K8S_NAMESPACE}"
                echo 'Helm Chart Deployed'
            }
        }
    }
}
