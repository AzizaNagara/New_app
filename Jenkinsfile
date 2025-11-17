pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'dockerraziza/new_app'  // Utilisez ici le nom d'image que vous voulez
        KUBE_NAMESPACE = 'default'
        DOCKER_HOST = 'npipe:////./pipe/docker_engine'
        HELM_CHART_PATH = './mon-app'
    }
    stages {
        stage('Cloner le dépôt') {
            steps {
                // Cloner la branche helm-deployment pour le déploiement avec Helm
                git branch: 'helm-deployment', url: 'https://github.com/AzizaNagara/New_app.git'
            }
        }
        stage('Construire l\'image Docker') {
            steps {
                script {
                    // Construire l'image Docker avec le tag correspondant
                    sh 'docker build -t $DOCKER_IMAGE .'
                }
            }
        }
        stage('Pousser l\'image Docker') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    // Se connecter à Docker Hub avec les identifiants
                    sh 'docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD'
                    // Pousser l'image sur Docker Hub
                    sh 'docker push $DOCKER_IMAGE'
                }
            }
        }
        stage('Déployer avec Helm') {
            steps {
                script {
                    // Déployer l'application avec Helm
                    sh 'helm upgrade --install mon-app $HELM_CHART_PATH --namespace $KUBE_NAMESPACE --set image.repository=$DOCKER_IMAGE --set image.tag=latest'
                }
            }
        }
    }
}

