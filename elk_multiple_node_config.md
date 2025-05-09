# CONFIGURATION  MULTISERVEUR POUR elasticsearch - Kibana et Filebeat (Beats)

<p>
  <img align="center" height=80 src="https://github.com/josoavj/ELK_Config/blob/master/assets/elastic-logo.png" alt="elastic"/>
  <img align="center" height=100 src="https://github.com/josoavj/ELK_Config/blob/master/assets/elastic-elasticsearch-logo.png" alt="elasticsearch"/>
  <img align="center" height=100 src="https://github.com/josoavj/ELK_Config/blob/master/assets/elastic-kibana-logo.png" alt="Kibana"/>
</p>



## 1 - Présentation

- Installation et configuration dans de multiples machines: deux au max (Mais peut s'appliquer à plus de trois noeuds)
- Structuration des serveurs: un pour elasticsearch et Kibana, et un autre pour les extensions 
- Les fonctions de chaque noeuds dans votre cluster dépend de vos préferences, mais vous devez au moins avoir ces principaux noeuds:
  - Un noeud Maître (Master Node)
  - Un noeud de données (Data Node)
  - Un noeud client servant de relais entre les deux noeuds
- Vous pouvez utiliser un seul noeud au début, cela dépend de votre préference mais ce n'est pas vraiment recommandé.
- Pour les systèmes d'exploitation basés sous Linux:
  - Installer via l'archive (.tar)
  - Ou installer directement en tant qu'application suivant les instructions données.
- Pour elasticsearch **V 8.x**: La sécurité est activée par défaut.
- Ce fichier comporte une configuration complète d'elasticsearch et de Kibana avec la licence Basique

### Outils utilisés pour l'application de ces configurations

- Deux Serveurs dediés: Ubuntu Server
  - Le premier pour elasticsearch et Kibana
  - Le second pour Filebeat
- SSH pour faciliter l'accès distant aux deux serveurs.
  - SSH utiliser le **port 22** et le protocole **TCP**.


### Remarque: 

- Dans l'ensemble du processus, vous aurez besoin de créer un compte elasticsearch si vous utilisez une distribution Linux:
  - Créer un utilisateur et un groupe d'utilisateur sous le nom **elasticsearch**
  - Création: `sudo adduser elasticsearch`
  - Ajouter l'utilisateur dans le groupe sudo: `sudo usermod -aG sudo elasticsearch`
  - Vérifier si l'utilisateur est créé via: `cat /etc/passwd`
  - Changer vers elasticsearch: `su elasticsearch`
- Executez les requêtes et commandes par cet utilisateur
- De même pour Kibana et les autres services: créer un utilisateur kibana ou lancez-le avec l'utilisateur elasticsearch
- Cette méthode vous sauvera de divers problème sur les autorisations et les comptes
- Bien qu'il soit recommandé d'utiliser l'archive comme moyen d'installation sous Linux, si vous utilisez un Distro basé sur Debian; je recommande d'utiliser le pachage .deb
- Vous pouvez le consulter dans le site officiel d'**[elastic](https://www.elastic.co/guide)**

## 2 - Installation globale pour elasticsearch - Kibana et Filebeat

### Utilisation de l'archive (.tar) comme moyen d'installation

- Installer suivant votre configuration 
```
if (installation == linux){
  
  lier_au_path(ELK_PATH);

  } else {
  
  go_to(configuration);
  
  }
```
- Lier au path: 
  - Créer un fichier `.extension_nom_terminal`: `.extension_bash` or `.extension.fish`
  - nano `.extension_nom_terminal`
  - Pour fish: 
    - `set -gx NOMPATH0 /path/to/bin`
    -  `set -gx PATH $PATH $NOMPATH0`
    - **-gx:** Pour une configuration globale
  
  - Pour bash:
    - `export NOMPATH0=/path/to/bin`
    - `export PATH=$PATH:$NOMPATH0`
  - Recommandé: au cas ou il y a au moins deux terminal, veuillez configurer les deux
  - Ensuite: `source .extension_nom_terminal`

## 3 - Installation et initialisation d'elasticsearch

- Si vous voulez lancer une instance sans configurer, vous pouvez juste commencer ici
- Mais si vous voulez configurer votre cluster et noeud elasticsearch, passez d'abord par le changement de configuration
- Run: `./../bin/elasticsearch` [Pour une configuration basique]
- Run with systemd (Recommended on Linux Based server):
     - `sudo systemctl enable elasticsearch.service` 
     - `sudo systemctl start elasticsearch.service`

### Compte elasticsearch

- Il existe plusieurs comptes par défaut dans elasticsearch, tel que **elastic**. Chaque compte possède leurs propres rôles.
- Chaque rôle définira le privilège des utilisateurs. Pour **elastic**, il a le rôle du superutilisateur (**Superuser**)
- elasticsearch fonctionnera sans mot de passe utilisateur. Mais pour la sécurité de votre serveur, vous devez créer votre propre mot de passe.
- Pour ajouter ou changer/réinitialiser le mot de passe d'un compte, lancez la commande:  `elasticsearch-reset-password -i -u username`
     - **-i:** interactive (Pour une session interactive)
     - **-u:** votre nom d'utilisateur
     - On va vous demander de créer puis de confirmer le nouveau mot de passe. 
- Ou procéder par: `elasticsearch-setup-password interactive` (Mais pour cette commande, on vas vous demander de créer des mots de passe pour tous les services d'elasticsearch)
  [Au cas ou la sécurité par défaut est defaillant ou n'est pas activée]
- Vous pouvez aussi ajouter de nouveau utilisateur ayant des rôles spécifiques pour votre serveur elasticsearch.
- Utilisez la commande suivante pour cela: `elasticsearch-users useradd nouveau_utilisateur -r superuser`
  - elasticsearch va vous demander d'entrer un mot de passe pour le compte par la suite.
  - **-r ou --rôles** permet de définir un rôle pour l'utilisateur.
    
#### Les rôles prédefinis ou Built-in Roles dans elasticsearch

| Rôle Prédéfini                  | Description                                                                                                |
| :------------------------------ | :--------------------------------------------------------------------------------------------------------- |
| `superuser`                     | Accès complet à toutes les opérations du cluster et des index. **À utiliser avec grande prudence.** |
| `kibana_user`                   | Accès général aux fonctionnalités de Kibana pour visualiser et interagir avec les données.                 |
| `kibana_dashboard_only_user`    | Accès en lecture seule aux tableaux de bord Kibana.                                                      |
| `monitoring_user`               | Accès aux données de monitoring du cluster (métriques, logs, etc.).                                      |
| `apm_user`                      | Rôle pour les utilisateurs d'Elastic APM (Application Performance Monitoring).                             |
| `apm_system`                    | Rôle système pour les opérations internes d'Elastic APM.                                                  |
| `beats_admin`                   | Rôle pour l'administration des configurations des Beats.                                                  |
| `beats_system`                  | Rôle système pour les opérations internes des Beats.                                                     |
| `logstash_admin`                | Rôle pour l'administration des configurations de Logstash.                                                |
| `logstash_system`               | Rôle système pour les opérations internes de Logstash.                                                    |
| `machine_learning_admin`        | Rôle pour l'administration des fonctionnalités de Machine Learning.                                         |
| `machine_learning_user`         | Rôle pour l'utilisation des fonctionnalités de Machine Learning.                                          |
| `data_admin`                    | Permet d'effectuer des opérations d'administration sur les données (création/suppression d'index, etc.). |
| `data_reader`                   | Permet de lire les données des index.                                                                     |
| `ingest_admin`                  | Rôle pour la gestion des pipelines d'ingestion.                                                            |
| `ingest_user`                   | Rôle pour l'utilisation des pipelines d'ingestion.                                                            |
| `snapshot_admin`                | Rôle pour la gestion des snapshots d'index.                                                                 |
| `snapshot_user`                 | Rôle pour l'utilisation des snapshots d'index.                                                                 |
| `remote_monitoring_agent`       | Rôle pour les agents de monitoring de clusters distants.                                                 |
| `remote_monitoring_collector`   | Rôle pour le collecteur de données de monitoring de clusters distants.                                     |

- Pour voir les comptes présents dans votre serveur, executez: `elasticsearch-users list`
  
### Confirmation et vérification

- Ensuite: sudo systemctl status elasticsearch [Vérifier si elasticsearch est fonctionnel]
- Après: curl --cacert /etc/elasticsearch/certs/http_ca.crt -u elastic https://localhost:9200
- Ce dernier est fait pour vérifier si elasticsearch est fonctionnel sur votre serveur

### Changer la configuration pour elasticsearch

- GoTo `/etc/elasticsearch/elasticsearch.yml`:
  - Faire une copie de votre fichier avant tout elasticsearch.yml.backup 
  - Changer les configurations suivantes:
     - `cluster.name: mon-application`
     - `node.name: node-1`
     - `network.host: 172.27.28.15` (Adresse IP de l'hôte)
     #### Avoir son adresse IP: Sur Linux, run: ifconfig
     - `http.port: 9200` (Port à utiliser)

### Issue: Problème de démarrage du service elasticsearch

- Erreur de démarrage du elasticsearch.service
- GoTo: `/usr/lib/systemd/system/`
- Run: `nano elasticsearch.service`
- Si les informations suivantes ne correspondent pas à votre environnement, veuillez les changer:
    - `User=username`
    - `Groupe=group`
- Run: `sudo systemctl daemon-reload`
- Run: `sudo systemctl status elasticsearch`
- Run: `sudo systemctl enable elasticsearch`
- Run: `sudo systemctl start elasticsearch`

### Autre solution (Si vous n'avez pas opté la suggestion au début)

- Créer un utilisateur et un groupe d'utilisateur sous le nom **elasticsearch**
- Création: `sudo adduser elasticsearch`
- Ajouter l'utilisateur dans le groupe sudo: `sudo usermod -aG sudo elasticsearch`
- Vérifier si l'utilisateur est créé via: `cat /etc/passwd`
- Changer vers elasticsearch: `su elasticsearch`
- Executez les requêtes et commandes par cet utilisateur

## 4 - Pour kibana 

- Installer kibana 
- Vérifier l'intégrité de tous les fichiers dans les dossiers suivants:
  - `/usr/share/kibana/`  
  - `/etc/kibana/`
  - `/var/lib/kibana`
- Vérifier l'integrité de kibana avec: `kibana-integrite-check`
- Créer un token elasticsearch pour kibana: `elasticsearch-create-enrollement-token -s kibana`
- Ce token vous permet d'intégrer faciement Kibana dans elasticsearch
- Run: `kibana-setup`
- Ensuite run: `kibana-encryption-keys generate`
- Copier les clés générés dans kibana.yml
- **Remarque:** Ne lancez pas Kibana qu'après ces étapes mentionnés 

- Lancement de Kibana en tant que processus:
  - Run: `sudo systemctl daemon-reload`
  - Run: `sudo systemctl enable kibana`
  - Run: `sudo systemctl start kibana`
 
- Pour configurer Kibana selon vos besoins: 
  - Allez vers: `/etc/kibana/`
  - Exécutez : `cp kibana.yml kibana.yml.backup`. Cela va vous permettre de ne pas perdre l'ancienne version du fichier.
  - Then `nano kibana.yml`
- Dans le fichier de configuration **kibana.yml**: 
  ```
  server.port: 5601                                                                 # Le port à utiliser pour Kibana
  server.host: "kibana_IP"
  elasticsearch.hosts: ["https://localhost:9200"]
  elasticsearch.username: "kibana_system"
  elasticsearch.password: "ur_passwd"
  elasticsearch.ssl.certificateAuthorities: ["/etc/kibana/certs/http_ca.crt"]        # A copier venant de /etc/elasticsearch/certs
  server.publicBaseUrl: "http://kibana_ip:5601"                                     # Pour une utilisation standard
  ```

### Tips: Copie vers un autre serveur ssh (Via Secure Copy)

- `mkdir /etc/kibana/certs`
- Do: `scp /etc/elasticsearch/certs/http_ca.crt sfi@ip:/etc/kibana/certs`
- Ensuite: `elasticsearch-reset-password -i -u kibana_system` [Definition du mot de passe kibana]
- Rédemarrer le service de Kibana: `sudo systemctl restart kibana.service`

### ADS: Ajout automatique d'un nouveau node elasticsearch vers un cluster existant

- La version d'elasticsearch pour chaque node doivent être les mêmes
- Les noeuds ne doivent pas êtres hébergés dans le même serveur.
- Le noeud Maître est celui qui lance le cluster.
- Veuillez suivre les mêmes instructions que le premier noeud pour les autres sauf pour certaines configurations.
- Configuration dans **../../elasticsearch.yml** de votre elk:
   ```
   cluster.name: same_as_master
   node.name: node-x
   host: votre_hôte
   port: votre_port
   ```
- Dans le noeud Maître: `elasticsearch-create-enrollment-token -s node`
- Copier le token elasticsearch generé
- Executer la commande suivant dans le nouveau serveur node elasticsearch:
   - `elasticsearch-reconfigure-node --enrollement-token votre_token`
- Ensuite `systemctl daemon-reload` dans le nouveau serveur elasticsearch
- Verifier dans kibana si les changement sont pris en compte (Nombre de nodes)
- Done xD
- **Remarque:** Il est recommandé de bien enroller tous vos nodes dans un cluster avant de sécuriser ce dernier
- Je suggère aussi d'activer par défaut le monitoring dans kibana pour mieux suivre les activités dans votre cluster
- Vous pouvez l'activer facilement dans l'interface de Kibana

## 5 - Génerer un certificat pem pour votre cluster elasticsearch

- Les certificats sont utiles pour une connexion sécurisée entre les instances et services
- Run `./bin/elasticsearch-certutil ca` ou `elasticsearch-certutil ca` 
- Cela vous donnera le fichier:  **elastic-stack-ca.p12** [Sécurisé avec un mot de passe]
- Ensuite lancez: `elasticsearch-certutil cert --ca elastic-stack-ca.p12` [Pour génerer un clé et un certificat privé pour votre node (Single node)]
- Cela vous donnera un  fichier: **elastic-certificates.p12**
- Ajouter les configurations suivantes dans **elasticsearch.yml**:
  ```
  xpack.security.transport.ssl.enabled: true
  xpack.security.transport.ssl.client_authentication: required
  xpack.security.transport.ssl.verification_mode: certificate
  xpack.security.transport.ssl.keystore.path: /path/to/certs/elastic-certificates.p12
  xpack.security.transport.ssl.truststore.path: /path/to/certs/elastic-certificates.p12
  ```
- Lance:  `sudo chown root elastic-certificates.p12`
- Ensuite lance: `sudo chmod 660 elastic-certificates.p12` {Pour autoriser la lecture et écriture dans le fichier}
  - **6:** Lecture et écriture pour l'utilisateur actuel
  - **6:** Lecture et écriture pour le groupe contenant l'utilisateur
  - **0:** Aucune permission accordée aux autres utilisateurs
- Lance par la suite :
  - `elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password`
  - `elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password`
  - Cela ajoutera les mot de passe du transport ssl dans le Keystore elasticsearch
  
- **Remarque:** copiez le fichier **elastic-certificates.p12** et suivre les mêmes instructions dans chaque node de votre cluster elasticsearch.
- Ensuite: `keytool -importcert -trustcacerts -noprompt -keystore elastic-stack-ca.p12 \ -storepass <password>  -alias new-ca -file ca.crt`
- Vérifier la liste de certificat avec: `keytool -keystore config/elastic-stack-ca.p12 -list`


### Tips (Si votre utilisateur n'est pas elasticsearch)

- Au cas où elasticsearch ne peut pas lire ou écrire le contenu de votre dossier/fichier, passer en ces commandes: 
  - `sudo chown user:group filename`
  - `sudo chown -R user:group folder` 
    - Ils permettent de changer l'utilisateur propriétaire et le groupe d'utilisateur de votre fichier ou dossier
  - `sudo chmod 660 file_name`
     -  Permet de changer la permission sur la lecture et écriture du document par l'utilisateur concerné

## 6 - Génerer un certificat pour securiser la connexion entre chaque node

- Ensuite pour un certificat pour chaque node: `elasticsearch-certutil http`
- Suivre les instructions données
- Pour génerer des nouveaux certificats pour toutes vos nodes:
  - Instructions:
       - CSR: N[Non]
       - CA existant : Y [Yes]
       - Path du CA [.../elastic-stack-ca.p12]
       - Entrer le mot de passe du CA
       - Entrer la valeur d'expiration de votre certificat
       - Génerer un certificat par node: Y [Yes]
       - Répartition de chaque configuration:
         - Chaque node possède son propre clé privée
         - Un hostname spécifique
         - Une adresse IP spécifique
         - Après avoir généré vos certificats, veuillez configurer un mot d passe pour votre keystore
         - Vous obtenez un fichier elasticsearch-ssl-http.zip
         - Extraire votre fichier: elle doit contenir deux dossiers: "elasticsearch" et "kibana"
         - Dans kibana se trouve le fichier elasticsearch-ca.pem qui vas être utilisé pour la configuration vers kibana
         - Configuration de kibana [kibana.yml]: elasticsearch.ssl.certificateAuthorities: KBN_PATH_CONF/certs/elasticsearch-ca.pem
         - Structure basique pour chaque dossier dans elasticsearch en fonction des noms de vos nodes:
           - node_name
             - README.txt
             - http.p12
             - sample-elasticsearch.yml
         - Copier http.p12 dans certs/
         - Run :  `elasticsearch-keystore add xpack.security.http.ssl.keystore.secure_password`

## 7 - Securisation de la connexion de kibana avec le serveur

- Run `elasticsearch-certutil csr -name kibana-server` [Obsolète, passer à l'etape suivante]
- Ensuite lancer: `elasticsearch-certutil cert --pem -ca /usr/share/elasticsearch/elastic-stack-ca.p12  -name kibana-server`
- Copier le fichier généré **kibana-server.crt** vers **/etc/kibana/certs**
- De même pour kibana-server.key
- Lier ces fichiers dans **kibana.yml**:
  ```
  server.ssl.enabled: true
  server.ssl.certificate: /etc/kibana/certs/kibana-server.crt
  server.ssl.key: /etc/kibana/certs/kibana-server.key
  ```
- Cela changera le protocole de votre serveur kibana en https



## 8 - Self monitoring avec Kibana (Activation)

- Activer le monitoring pour une instance Kibana
- {Activation du self-monitoring dans kibana}
- Il existe deux méthodes pour cela: 
  - Kibana + Metricbeat (Recommandé)   
  - Legacy collectors
- Ici on utilisera la **legacy collectors**
- De base, on a ceci par défaut: `monitoring.kibana.collection.enabled: false`
- Cela peut se faire avec la licence basique
- Aller dans **elasticsearch.yml** et ajouter:
  - `xpack.monitoring.collection.enabled: true`
    - Vous pouvez le faire pour toutes vos nodes
- Ensuite dans **kibana.yml**:
  - `monitoring.kibana.collection.enabled: true`
- Vous pouvez voir dans Kibana que le self monitoring pour votre cluster est activé



## 9 - Activer le monitoring avec metricbeat

- Pour tracer l'activité de votre cluster et ses nodes
- Aller dans kibana **Management > Stack-management**
- Si Metricbeat non installé, veuillez l'installer (Via le lien fourni dans kibana)
- Changer le mots de passe pour **beats_system** dans Kibana ou `elasticsearch-reset-password -i -u beats_system`
- Lancer:  `metricbeat modules list`
- En cas d'erreur: `metricbeat modules enable elasticsearch-xpack`
- Remarque: Ajouter metricbeat dans le même serveur qu'elasticsearch

## 10 - Configuration de fleet server

- Protocole: `TCP 8220 > Agent (TCP 8820 To the Agent)`
- Déploiement en local pas dans le cloud (On premises)
- On ajoute le serveur fleet et le serveur elasticsearch
- Configuration valide pour la version 7.13 et plus
- Prérequis:
  - Avoir un CA (Certificat Authority)
- Dans votre serveur Kibana: 
  - Aller dans Menu > Fleet
  - Aller dans Settings (Fleet)
  - Ajouter vos serveurs elasticsearch (Host) {https://localhost:9200}
  - Passer à : Add Fleet server
  - Add Fleet server > Advanced > Production > Add fleet server Host (Port 8220) > Installing from a centralized host
  - Ouvrir un terminal et executer: `elasticsearch-certutil cert --pem -ca /path/to/elastic-stack-ca.p12 -name fleet-server`
  - On obtient les fichier suivant: `fleet-server.crt fleet-server.key`
  - Copier ces fichiers dans le dossier pour fleet (../fleet/certs/)
  - Veuiller utiliser la même CA que Kibana dans fleet (elasticsearch-ca.pem)
  - Install Fleet 
- Continue enrolling agent with fleet

## 11 - Configuration d'un agent elastic (Elastic AGENT)

- On ne peut pas installer un agent dans un firewall fortigate
- Tous les agents doivent être connecté à l'hôte (Host)
- Aller dans **Add agent**
- Fonctionnement: 
  - Chaque agent possède un Policy
  - Chaque agent gère une integration
- Enroll with fleet **enabled**
- Utiliser linux tar pour ceci
- Copier la commande donnée
- Dans terminal, ajouter `--certificate-authorities=../path/certs/elasticsearch-ca.pem --insecure`
```
# Si elasticsearch reçoit des données alors votre agent est installé correctement
If (incoming-data) {
  Agent_successfully installed
  } 
```
- Une fois ces configurations (Fleet & Agent) vous pouvez ajouter des intégrations

