pipeline {
    agent any
    tools {
        maven 'maven3.6.3'
    }
    stages {
        stage('Clean') {
            steps {
                sh 'mvn clean'
            }
        }
        stage('Package') {
            steps {
                sh 'mvn package -P docker'
            }
        }
        stage('Push') {
            steps {
                withEnv(['docker=/usr/local/bin/docker']) {
                    sh '$docker image tag k8s/01authserver:1.0-SNAPSHOT ${MINIKUBE_REGISTRY}/01authserver:${BUILD_NUMBER}'
                    sh '$docker push ${MINIKUBE_REGISTRY}/01authserver:${BUILD_NUMBER}'
                    sh '$docker image tag k8s/02resourceserver:1.0-SNAPSHOT ${MINIKUBE_REGISTRY}/02resourceserver:${BUILD_NUMBER}'
                    sh '$docker push ${MINIKUBE_REGISTRY}/02resourceserver:${BUILD_NUMBER}'
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                     def inputText = readFile "./k8s/oauthserver-deployment.yaml"
                     inputText = inputText.replaceAll("<PARAM_REGISTRY_IMAGE>", "${MINIKUBE_REGISTRY}/01authserver:${BUILD_NUMBER}")
                     writeFile file: "./k8s/oauthserver-deployment.yaml", text: inputText

                     inputText = readFile "./k8s/resourceserver-deployment.yaml"
                     inputText = inputText.replaceAll("<PARAM_REGISTRY_IMAGE>", "${MINIKUBE_REGISTRY}/02resourceserver:${BUILD_NUMBER}")
                     writeFile file: "./k8s/resourceserver-deployment.yaml", text: inputText

                }
                withEnv(['docker=/usr/local/bin/docker']) {
                    sh 'curl -X PUT ${MINIKUBE_API_SERVER}/apis/apps/v1/namespaces/default/deployments/oauthserver --header "Authorization: Bearer ${MINIKUBE_TOKEN}" -H "Content-Type: application/yaml" --data-binary @"k8s/oauthserver-deployment.yaml" --insecure'
                    sh 'curl -X PUT ${MINIKUBE_API_SERVER}/apis/apps/v1/namespaces/default/deployments/resourceserver --header "Authorization: Bearer ${MINIKUBE_TOKEN}" -H "Content-Type: application/yaml" --data-binary @"k8s/resourceserver-deployment.yaml" --insecure'
                }
            }
        }
    }
}