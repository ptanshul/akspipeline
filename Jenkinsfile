pipeline {
    agent any

    environment {
        ACR_NAME = "cloudacr"
        AKS_RESOURCE_GROUP = "cloudrg"
        AKS_CLUSTER_NAME = "cloudaks"
        IMAGE_NAME = "flask-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${ACR_NAME}.azurecr.io/${IMAGE_NAME}:${IMAGE_TAG}")
                    
                }
            }
        }

    

        stage('Deploy to AKS') {
            steps {
                withCredentials([azureServicePrincipal('44aa038a-cc50-4b8d-b99c-306019316abf')]) {
                    sh """
                    az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
                    az aks get-credentials --resource-group $AKS_RESOURCE_GROUP --name $AKS_CLUSTER_NAME
                    
                    sed -i 's|${ACR_NAME}.azurecr.io/${IMAGE_NAME}:.*|${ACR_NAME}.azurecr.io/${IMAGE_NAME}:${IMAGE_TAG}|' kubernetes-deployment.yaml
                    kubectl apply -f kubernetes-deployment.yaml
                    """
                }
            }
        }
    }
}