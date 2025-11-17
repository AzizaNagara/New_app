pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'dockerraziza/new_app'  // Utilisez ici le nom d'image que vous voulez
        KUBE_NAMESPACE = 'default'
        DOCKER_HOST = 'npipe:////./pipe/docker_engine'
    }
    stages {
        stage('Cloner le dépôt') {
            steps {
                // Spécifier explicitement la branche main pour le clonage
                git branch: 'main', url: 'https://github.com/AzizaNagara/New_app.git'
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
        stage('Déployer sur Kubernetes') {
            steps {
                script {
                    // Appliquer les fichiers YAML pour le déploiement et le service Kubernetes
                    sh 'kubectl apply -f deployment.yaml --validate=false'
                    sh 'kubectl apply -f service.yaml --validate=false'
                }
            }
        }
    }
}

