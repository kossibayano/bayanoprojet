# Infrastructure & Déploiement Continu (MediaWiki / Jenkins / Ansible / Zabbix)

## Description
Projet d'automatisation d'infrastructure et d'intégration continue :
- **Orchestration / CI-CD :** Jenkins
- **Gestion de configuration :** Ansible
- **Infrastructure :** Serveurs Web (`http1`) et Base de données (`bdd1`)
- **Supervision :** Zabbix

## Architecture
- `ansible/` : Playbooks et configurations pour le déploiement de MediaWiki.
- `zabbix/` : Configurations pour la supervision.
- `Jenkinsfile` : Pipeline de déploiement automatique.
