pipeline {
    agent any

    /* --- VARIABLES D'ENVIRONNEMENT --- */
    environment {
        // Paramètres Zabbix
        ZABBIX_URL   = 'http://192.168.1.82/zabbix/' // Ajuste le chemin selon ton installation Zabbix
        ZABBIX_TOKEN = 63f59e19f0ab6c705a4352b1c41eb9748259c52098f83beddb36d289eafcc6d2''           // Remplace par ton token API Zabbix
        ZABBIX_HOST  = 'Jenkins-Server'                             // Nom exact de l'hôte dans Zabbix

        // Paramètres SSH / Ansible
        ANSIBLE_USER = 'user-ansible'                               // Ton utilisateur sur la VM Ansible
        ANSIBLE_IP   = '192.168.1.154'                               // IP de la VM Ansible Control Node
        ANSIBLE_DIR  = '/home/user_ansible/deploy_mediawiki'
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
                echo '=== Exécution du Playbook sur http1 et bdd1 ==='
                sh '''
                    ssh ${ANSIBLE_USER}@${ANSIBLE_IP} "cd ${ANSIBLE_DIR} && ansible-playbook -i inventaire.ini install-mediawiki.yml"
                '''
            }
        }

        stage('5. Healthcheck HTTP') {
            steps {
                echo '=== Test de réponse du serveur Web http1 ==='
                sh 'curl -sI http://http1 | grep "200 OK"'
            }
        }
    }

    /* --- NOTIFICATION À ZABBIX VIA HTTP REQUEST --- */
    post {
        success {
            echo '=== SUCCESS : Envoi du statut OK (0) à Zabbix ==='
            httpRequest httpMode: 'POST',
                        contentType: 'APPLICATION_JSON',
                        url: "${ZABBIX_URL}",
                        customHeaders: [[name: 'Authorization', value: "Bearer ${ZABBIX_TOKEN}"]],
                        requestBody: """{
                            "jsonrpc": "2.0",
                            "method": "history.push",
                            "params": [
                                {
                                    "host": "${ZABBIX_HOST}",
                                    "key": "jenkins.build.status",
                                    "value": "0"
                                }
                            ],
                            "id": 1
                        }""",
                        validResponseCodes: '200'
        }
        failure {
            echo '=== FAILURE : Envoi du statut ERROR (1) à Zabbix ==='
            httpRequest httpMode: 'POST',
                        contentType: 'APPLICATION_JSON',
                        url: "${ZABBIX_URL}",
                        customHeaders: [[name: 'Authorization', value: "Bearer ${ZABBIX_TOKEN}"]],
                        requestBody: """{
                            "jsonrpc": "2.0",
                            "method": "history.push",
                            "params": [
                                {
                                    "host": "${ZABBIX_HOST}",
                                    "key": "jenkins.build.status",
                                    "value": "1"
                                }
                            ],
                            "id": 1
                        }""",
                        validResponseCodes: '200'
        }
    }
}
