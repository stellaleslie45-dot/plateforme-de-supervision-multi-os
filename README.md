# plateforme-de-supervision-multi-os
Markdown

# 📊 Projet Supervision : Zabbix Server & Client Ubuntu

Ce dépôt contient la documentation technique et les fichiers de configuration nécessaires à la mise en place d'une infrastructure de supervision centralisée. L'objectif est de collecter, analyser et afficher les métriques d'un poste client Linux (Ubuntu) sur un serveur de supervision dédié, tout en gérant un système d'alerte automatique.

---

## 🛠️ Architecture de la Maquette

L'infrastructure est entièrement virtualisée sous VMware Workstation :

* **Serveur de Supervision :** Ubuntu Server 24.04 LTS (IP : `192.168.0.100`)
  * *Services :* Zabbix Server 7.0 LTS, Base de données, Grafana (Port `:3000`)
* **Poste Client Supervisé :** Ubuntu Client (IP : `192.168.0.150`)
  * *Service :* Zabbix Agent (Démon natif Linux)

---

## 🚀 Déploiement et Configurations

### 1. Installation et Configuration de l'Agent sur le Client Ubuntu

Pour remonter les indicateurs de performance (CPU, RAM, charge système, espace disque), l'agent Zabbix a été installé nativement sur la machine cliente :

```bash
# Mise à jour des dépôts et installation de l'agent
sudo apt update
sudo apt install zabbix-agent -y
```

Modification du fichier de configuration :

Les directives réseau ont été éditées dans le fichier de configuration principal /etc/zabbix/zabbix_agentd.conf pour cibler notre serveur de supervision :
Ini, TOML

Server=192.168.0.100
ServerActive=192.168.0.100
Hostname=Client-Ubuntu

Démarrage et persistance du démon Linux :

```Bash

# Rechargement, activation au démarrage et lancement de l'agent
sudo systemctl daemon-reload
sudo systemctl enable zabbix-agent
sudo systemctl restart zabbix-agent
```

2. Provisionnement de l'Hôte sur la Console Zabbix

L'enregistrement du poste client a été déclaré manuellement dans l'interface d'administration web de Zabbix (Data collection ➡️ Hosts) :

    Host name : Client-Ubuntu (Correspondance stricte et obligatoire avec le fichier de configuration).

    Templates associés : Linux by Zabbix agent.

    Host groups : Intégration dans le groupe générique Virtual machines.

    Interfaces : Ajout d'une interface réseau de type Agent sur le port standard 10050 avec l'adresse IP du client (192.168.0.150).

    Résultat : Après un cycle de scrutation, l'indicateur de disponibilité ZBX est passé au vert vif, confirmant la bonne communication.

3. Restitution Visuelle et Graphique sur Grafana

Couplage de l'interface de visualisation Grafana (Port :3000) avec l'API du serveur Zabbix pour la création de tableaux de bord de métrologie :

    Importation du Dashboard : Utilisation des modèles de structure au format JSON depuis le catalogue officiel de Grafana Labs (Plugin Zabbix en version 6.4.0).

    Variables de filtrage : Configuration des menus déroulants globaux pour cibler dynamiquement le groupe Virtual machines et isoler le comportement du Client-Ubuntu (Analyse sur un intervalle de Last 15 minutes).

🔔 Gestion des Alertes et Incidents (Mailto)

Une politique de notification automatique par e-mail a été implémentée pour alerter les administrateurs en cas d'anomalie sur le parc Linux.
Configuration du flux réseau :

    Media Type : Déclaration du relais SMTP sortant ciblant les serveurs de Google (smtp.gmail.com sur le port sécurisé 587 avec chiffrement STARTTLS).

    User Profile : Routage des alertes vers l'adresse de l'administrateur (stellaleslie45@gmail.com) reliée au compte Admin.

    Trigger Action : Activation de la règle automatique d'administration Report problems to Zabbix administrators.

Scénario de Test et Diagnostic :

Pour valider la chaîne d'alerte, un arrêt brutal du service a été provoqué directement sur le client Ubuntu :
```Bash
sudo systemctl stop zabbix-agent
```

    Constat dans l'Action Log : L'alerte "Zabbix agent is unavailable" s'est déclenchée instantanément dans l'interface.

    Analyse de l'audit technique : Les logs système affichent un statut Failed (Connection timed out). Ce comportement valide l'exactitude de la logique de l'infrastructure Zabbix. L'échec d'envoi physique du mail est uniquement lié aux restrictions de sécurité du pare-feu académique de l'école qui bloque les flux SMTP sortants sur le port 587.
