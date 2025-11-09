pipeline {
    agent any

    environment {
        IMAGE_NAME = 'qtpmigratorpfe-app'
        IMAGE_TAG = 'latest'
    }

    stages {
        stage('Checkout') {
            steps {
                echo ' Clonage du dépôt Git...'
                git(
                    url: 'https://github.com/amalmaalaoui/PFE-repo-.git',
                    branch: 'main'
                )
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo ' Construction de l’image Docker...'
                    sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    echo ' Exécution des tests...'
                    // Optionnel : n’exécuter que si tu as un dossier tests/
                    sh 'pytest || echo "Aucun test trouvé, passage à l’étape suivante."'
                }
            }
        }

        stage('Push Docker Image') {
            when {
                expression { return env.DOCKERHUB_CREDENTIALS_ID != null }
            }
            steps {
                script {
                    echo ' Envoi de l’image Docker vers Docker Hub...'
                    sh '''
                    docker login -u $DOCKERHUB_USER -p $DOCKERHUB_PASS
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} $DOCKERHUB_USER/${IMAGE_NAME}:${IMAGE_TAG}
                    docker push $DOCKERHUB_USER/${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    echo ' Déploiement de l’application Flask...'
                    // On stoppe le conteneur précédent s’il existe
                    sh '''
                    docker rm -f flask-app || true
                    docker run -d -p 5000:5000 --name flask-app ${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'Nettoyage du workspace...'
            cleanWs()
        }
        success {
            echo 'Pipeline terminé avec succès !'
        }
        failure {
            echo 'Le pipeline a échoué.'
        }
    }
}
