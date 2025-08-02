# CONFIGURATION  MULTISERVEUR POUR elasticsearch - Kibana et Filebeat (Beats)

<p>
  <img align="center" height=80 src="https://github.com/josoavj/ELK_Config/blob/master/assets/elastic-logo.png" alt="elastic"/>
  <img align="center" height=100 src="https://github.com/josoavj/ELK_Config/blob/master/assets/elastic-elasticsearch-logo.png" alt="elasticsearch"/>
  <img align="center" height=100 src="https://github.com/josoavj/ELK_Config/blob/master/assets/elastic-kibana-logo.png" alt="Kibana"/>
</p>

-----

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

-----

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

-----

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

-----

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

-----

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

-----

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
           ```
           /elasticsearch/
           ├── node_name/
           │   ├── README.txt
           │   ├── http.p12
           │   └── elasticsearch.yml
           └── Autres noeuds
           ```
           -----
          * **`README.txt`**: Un fichier texte simple qui peut contenir des informations importantes spécifiques à ce nœud. Par exemple :
            * Le rôle du nœud (maître, données, ingestion).
            * La version d'Elasticsearch installée.
            * Des notes sur la configuration ou des commandes spécifiques.
            * La date de la dernière modification.
          * **`http.p12`**: Ce fichier est un **keystore PKCS#12** qui contient à la fois le certificat SSL/TLS du nœud (souvent un certificat généré par votre CA pour la communication HTTP) et sa clé privée correspondante. Il est utilisé par Elasticsearch pour sécuriser les communications HTTP entrantes et sortantes de ce nœud.
          * **`elasticsearch.yml`**: Le fichier de configuration principal pour ce nœud Elasticsearch spécifique. Ce fichier contiendra tous les paramètres de configuration propres à ce nœud, y compris :
            * `node.name`: Le nom unique du nœud (doit correspondre au nom du dossier).
            * `path.data` et `path.logs`: Les chemins vers les répertoires de données et de logs du nœud.
            * `network.host`: L'adresse IP ou le nom d'hôte sur lequel le nœud écoute.
            * `cluster.name`: Le nom de votre cluster Elasticsearch.
          * Les configurations de sécurité (`xpack.security.http.ssl.*`) qui pointeront vers votre fichier `http.p12`.
         - Copier http.p12 dans certs/
         - Run :  `elasticsearch-keystore add xpack.security.http.ssl.keystore.secure_password`

-----

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

-----

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

-----

## 9 - Activer le monitoring avec metricbeat

- Pour tracer l'activité de votre cluster et ses nodes
- Aller dans kibana **Management > Stack-management**
- Si Metricbeat non installé, veuillez l'installer (Via le lien fourni dans kibana)
- Changer le mots de passe pour **beats_system** dans Kibana ou `elasticsearch-reset-password -i -u beats_system`
- Lancer:  `metricbeat modules list`
- En cas d'erreur: `metricbeat modules enable elasticsearch-xpack`
- Remarque: Ajouter metricbeat dans le même serveur qu'elasticsearch

-----

## 10 - Configuration de Fleet Server

Le **Fleet Server** est un composant crucial de l'architecture Elastic Agent. Il agit comme un point de coordination centralisé pour tous vos Elastic Agents, leur permettant de télécharger les stratégies, d'envoyer les données et de maintenir leur statut.

Voici un guide détaillé pour configurer votre Fleet Server :

  * **Protocole et Ports** :

      * Le Fleet Server écoute les connexions des Agents Elastic sur le port **TCP 8220**.
      * Les Agents Elastic se connectent au Fleet Server sur ce même port (ils *ne se connectent pas* sur le port 8820 ; c'est le port d'écoute du Fleet Server qui est pertinent).

  * **Type de Déploiement** :

      * Cette configuration se concentre sur un déploiement **On-premises** (sur vos propres serveurs), par opposition à un déploiement dans le cloud (Elastic Cloud).

  * **Connexions Essentielles** :

      * Pour fonctionner, le Fleet Server doit pouvoir communiquer avec votre serveur **Elasticsearch** (pour indexer les données et gérer les états des agents) et être accessible par les **Agents Elastic**.

  * **Compatibilité des Versions** :

      * Ces instructions sont valides pour les versions d'Elastic Stack **7.13 et ultérieures**, où Fleet Server a été introduit et est devenu le moyen principal de gérer les Agents Elastic.

  * **Prérequis Indispensables** :

      * **Autorité de Certification (CA)** : Il est **impératif** d'avoir une Autorité de Certification (CA) pour générer et signer les certificats SSL/TLS nécessaires à la communication sécurisée entre Fleet Server, Elasticsearch, Kibana et les Agents. Utiliser une CA commune pour tout l'écosystème Elastic Stack simplifie grandement la gestion des certificats et assure une communication chiffrée.

  * **Étapes de Configuration dans Kibana** :

    1.  **Accéder aux Paramètres Fleet** :

          * Dans votre interface Kibana, naviguez vers le **Menu principal**.
          * Recherchez et cliquez sur la section **Fleet**.
          * Dans la section Fleet, allez dans les **Settings** (Paramètres).

    2.  **Ajouter l'Hôte Elasticsearch** :

          * Sous "Elasticsearch hosts", ajoutez l'URL de votre serveur Elasticsearch. Par exemple : `https://localhost:9200`. Si Elasticsearch est sur une autre machine, remplacez `localhost` par son adresse IP ou son nom d'hôte.

    3.  **Ajouter un Fleet Server** :

          * Cliquez sur **Add Fleet server**.
          * Sélectionnez l'option "Advanced" ou "Production" (selon la version de Kibana, l'interface peut varier légèrement).
          * Choisissez "Add fleet server Host" et spécifiez l'adresse de l'hôte sur lequel votre Fleet Server va s'exécuter, en incluant le port par défaut (8220). Exemple : `votre-serveur-fleet:8220`.
          * Sélectionnez "Installing from a centralized host" ou l'option équivalente qui indique que vous allez installer Fleet Server manuellement sur une machine dédiée.

    4.  **Génération des Certificats pour Fleet Server** :

          * Depuis un terminal sur une machine ayant accès à votre CA et à `elasticsearch-certutil`, exécutez la commande suivante pour générer les certificats et la clé privée pour Fleet Server. Remplacez `/path/to/elastic-stack-ca.p12` par le chemin réel de votre fichier de CA :
            ```bash
            elasticsearch-certutil cert --pem -ca /path/to/elastic-stack-ca.p12 -name fleet-server
            ```
          * Cette commande produira deux fichiers essentiels : `fleet-server.crt` (le certificat public de Fleet Server) et `fleet-server.key` (la clé privée de Fleet Server).

    5.  **Copie des Certificats** :

          * Copiez ces deux fichiers (`fleet-server.crt` et `fleet-server.key`) dans le dossier désigné pour les certificats de Fleet Server sur la machine où il sera installé. Un emplacement courant est `../fleet/certs/` ou `[fleet-server-install-dir]/certs/`.
          * Assurez-vous également que la **même CA** que celle utilisée par Kibana et Elasticsearch (`elasticsearch-ca.pem`) est disponible pour Fleet Server. Elle doit être copiée dans le répertoire des certificats de Fleet Server pour qu'il puisse valider les connexions des agents et à Elasticsearch.

    6.  **Installation et Démarrage de Fleet Server** :

          * Kibana vous fournira des instructions spécifiques (une commande `install` ou `setup`) pour installer et démarrer le processus Fleet Server sur votre machine désignée. Suivez attentivement ces instructions. Celles-ci incluront des références aux certificats que vous venez de générer.

  * **Poursuivre l'Enrôlement des Agents** :

      * Une fois que votre Fleet Server est configuré et en ligne (vous pouvez vérifier son statut dans Kibana sous "Fleet" \> "Fleet Servers"), vous pouvez continuer à **enrôler vos Agents Elastic**. Les agents utiliseront ce Fleet Server comme leur point de contact central.

-----

## 11 - Configuration d'un Agent Elastic

Un **Agent Elastic** est un programme léger qui s'installe sur vos hôtes pour collecter des données et les envoyer à Elasticsearch. Il est géré de manière centralisée via **Fleet**, une interface dans Kibana.

Voici quelques points clés à retenir et des étapes pour le configurer :

  * **Où installer l'agent ?** Vous installez l'Agent Elastic sur les **hôtes** dont vous souhaitez collecter les données. Par exemple, sur des serveurs Linux, Windows, ou macOS.

      * **Restriction importante** : Vous ne pouvez généralement pas installer un Agent Elastic directement sur un pare-feu matériel comme un FortiGate, car ces appareils ont des systèmes d'exploitation propriétaires et fermés. Pour collecter des logs d'un FortiGate, vous utiliseriez plutôt une intégration spécifique via Syslog ou SNMP, où les logs sont envoyés à un hôte sur lequel un Agent Elastic est installé.

  * **Connexion de l'agent** : Tous les agents doivent être connectés à un **hôte** (un serveur ou une machine virtuelle) où ils peuvent s'exécuter. L'agent lui-même ne gère pas de connexions directes entre différents agents.

  * **Processus d'ajout d'un agent** :

    1.  Dans Kibana, naviguez vers la section **Fleet** (généralement sous "Management").
    2.  Cliquez sur **Add agent**.

  * **Fonctionnement de l'agent** :

      * Chaque agent est associé à une **stratégie (Policy)**. Une stratégie définit quelles intégrations sont activées sur cet agent et comment elles doivent fonctionner (par exemple, quels logs collecter, quelles métriques surveiller).
      * Chaque agent gère une ou plusieurs **intégrations**. Une intégration est un ensemble prédéfini de configurations pour collecter des données d'une source spécifique (par exemple, logs système, métriques de Docker, données de Nginx).

  * **Enrôlement avec Fleet** : Assurez-vous que l'option **Enroll with Fleet** est **activée** lors de la configuration de l'agent. Cela permet à Fleet de gérer l'agent de manière centralisée.

  * **Installation de l'agent (Exemple Linux Tarball)** :

    1.  Choisissez la méthode d'installation appropriée pour votre système d'exploitation. Pour Linux, l'utilisation d'un **tarball** (archive .tar.gz) est une option courante.
    2.  Kibana vous fournira une **commande spécifique** à copier et exécuter sur votre hôte. Cette commande inclut l'URL de Fleet et le jeton d'enrôlement.
    3.  **Gestion des certificats (important !) :** Si votre déploiement Elasticsearch utilise des certificats SSL/TLS personnalisés et que l'agent ne fait pas confiance à l'autorité de certification par défaut, vous devrez spécifier le chemin vers le certificat de l'autorité de certification Elasticsearch (CA). Ajoutez l'option `---certificate-authorities=/chemin/vers/votre/certs/elasticsearch-ca.pem` à la commande d'installation.
    4.  Si vous rencontrez des problèmes de validation de certificat pendant le développement ou les tests (non recommandé pour la production), vous pouvez temporairement ajouter `--insecure` à la commande, mais **ce n'est pas une pratique sécurisée pour les environnements de production.**

  * **Vérification de l'installation** :

    ```
    # Si Elasticsearch reçoit des données de votre agent,
    # c'est le signe que l'agent est correctement installé
    # et qu'il communique avec succès avec Fleet et Elasticsearch.
    ```

    Vous pouvez vérifier cela dans Kibana en allant dans "Fleet" et en regardant le statut de votre agent, ou en explorant les données entrantes dans "Discover".

  * **Après la configuration** : Une fois que Fleet et l'agent sont configurés et que l'agent est en ligne, vous pouvez commencer à **ajouter des intégrations** à la stratégie de l'agent. Chaque intégration permettra à l'agent de collecter un type de données spécifique.
