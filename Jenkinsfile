pipeline {
    agent any

    /* --- VARIABLES D'ENVIRONNEMENT --- */
    environment {
        // Paramètres SSH / Ansible
        ANSIBLE_USER = 'user-ansible'                               // Ton utilisateur sur la VM Ansible
        ANSIBLE_IP   = '192.168.1.154'                               // IP de la VM Ansible Control Node
        ANSIBLE_DIR  = '/home/user-ansible/deploy_mediawiki'
    }

    /* --- ÉTAPES DU PIPELINE (STAGES) --- */
    stages {

        stage('1. Checkout Git') {
            steps {
                echo '=== Récupération des sources Git ==='
                checkout scm
            }
        }

        stage('2. Sync vers VM Ansible') {
            steps {
                echo '=== Synchronisation des fichiers vers la VM Ansible ==='
                sh '''
                    ssh ${ANSIBLE_USER}@${ANSIBLE_IP} "mkdir -p ${ANSIBLE_DIR}"
                    rsync -avz --delete ansible/ ${ANSIBLE_USER}@${ANSIBLE_IP}:${ANSIBLE_DIR}/
                '''
            }
        }

        stage('3. Syntax Check Ansible') {
            steps {
                echo '=== Vérification de la syntaxe du Playbook ==='
                sh '''
                    ssh ${ANSIBLE_USER}@${ANSIBLE_IP} "cd ${ANSIBLE_DIR} && ansible-playbook -i inventaire.ini install-mediawiki.yml --syntax-check"
                '''
            }
        }

        stage('4. Déploiement MediaWiki') {
            steps {
                echo '=== Exécution du Playbook sur http2 et bdd1 ==='
                sh '''
                    ssh ${ANSIBLE_USER}@${ANSIBLE_IP} "cd ${ANSIBLE_DIR} && ansible-playbook -i inventaire.ini install-mediawiki.yml"
                '''
            }
        }

        stage('5. Healthcheck HTTP') {
            steps {
                echo '=== Test de réponse du serveur Web http2 ==='
                sh 'curl -sI http://http2 | grep "200 OK"'
            }
        }
    }

    /* --- NOTIFICATIONS EN CONSOLE --- */
    post {
        success {
            echo '=== SUCCESS : Déploiement effectué et validé avec succès ! ==='
        }
        failure {
            echo '=== FAILURE : Le déploiement a échoué ! ==='
        }
    }
}
