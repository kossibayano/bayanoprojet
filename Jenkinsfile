pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Récupération du code source depuis Git...'
            }
        }
        stage('Test') {
            steps {
                echo 'Exécution des tests automatisés...'
                sh 'python3 --version || node -v || java -version'
            }
        }
        stage('Build') {
            steps {
                echo 'Build réussi !'
            }
        }
    }
}
