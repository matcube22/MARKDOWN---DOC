Mise en place de la maquette du projet réseau de première année dans les hyperviseurs VMWare ESXI.
# Présentation de l'infrastructure
## Plan câblage physique
![schema physique](assets/schema_physique_virtualisation.png)
## Plan câblage "virtuel"
![schema virtuel](assets/schema_virtuel.png)

## Groupe de ports
Configuration du switch physique pour le taggage
```
Switch> enable
Switch# configure terminal
Switch(config)# interface gigabitethernet [Num ports]
Switch(config-if)# switchport mode trunk

```
## Routeur Cisco
```
Cisco> enable
Cisco# configure terminal
Cisco(config)# interface gigabitethernet [Num ports]
Cisco(config-if)# ip address @IP @MASK

# Configuration du routage
Cisco(config-if)# ip route add 0.0.0.0 0.0.0.0 172.16.0.1
Cisco(config)# ip routing

# Configuration pour le DHCP
Cisco(config-if)# ip helper-address 192.168.240.100

# Configuration des règles NAT et PAT
Cisco(config-if)# ip nat inside/outside # En fonction de l'interface si LAN ou WAN
Cisco(config)# access-list 1 permit 10.10.11.0 0.0.0.255
Cisco(config)# ip nat inside source liste 1 [interface sortie] overload
```
# Présentation des services
### Active Directory
[AJOUTER SCREENSHOT ADD]
### Web: Apache
[AJOUTER SCREENSHOT APACHE FONCTIONNEL]
## Services de supervisions
### GLPI
#### Installation
**Configuration: PHP**
La dernière version de GLPI nécessite php8.0, le dépôt ne contient que la version 7.0.
On commence par supprimer les paquets de php installés:
```bash
apt-get purge php7.*
```

Puis on nettoie les fichiers résiduels:
```
apt-get autoclean && apt-get autoremove
```

On doit ajouter le dépôt qui contient les paquets de php8.0
```
apt-add-repository "deb https://packages.sury.org/php/ buster main"
```

Si on fait un ``` apt update``` on tombera sur une erreur de sécurité où il manque la clé publique du dépôt.

Pour cela on ajoute la clé:
```bash
# Ajout de pgp pour ajouter la clé
apt install pgp

# Ajout de la clé du dépôt
wget -qO - https://packages.sury.org/php/apt.gpg |apt-key add -

# On met à jours nos dépôts
apt update
```

**Installation des paquets**
Mettez à jour la liste des **paquets** et les paquets eux-mêmes :
```bash
apt-get update && apt-get upgrade
```

Installation de **Apache**
```bash
apt-get install apache2 php libapache2-mod-php
```

Installation de **PHP** et de ces modules
```bash
apt-get install php8.0-imap php8.0-ldap php8.0-curl php8.0-xmlrpc php8.0-gd php8.0-mysql php8.0-cas php8.0-simplexml
```

**Installation de MariaDB**
```bash
apt-get install mariadb-server
```

Sécurisation de MariaDB
```bash
# Mot de passe root: vide
mysql_secure_installation
```

(Répondez “Y” à toutes les questions)

Concernant le mot de passe créé, c’est le compte root du MariaDB. N’oubliez pas de conserver votre mot de passe, nous en aurons besoin plus tard.

Installez les **modules complémentaires** au bon fonctionnement de GLPI.
```bash
apt-get install apcupsd php-apcu
```

**Redémarrez** les services :
```bash
/etc/init.d/apache2 restart
/etc/init.d/mysql restart
```

Créez la **base de données** qui nous permettra ensuite d’installer GLPI.
```bash
mysql -u root -p
```

Le mot de passe root est celui configurer plus haut.
```bash
MariaDB [(none)]> create database glpi; 
MariaDB [(none)]> grant all privileges on glpi.* to glpi@localhost identified by "lannion"; 
MariaDB [(none)]> quit
```

**Installation de GLPI**
On télécharge le fichier:
```bash
wget https://github.com/glpi-project/glpi/releases/download/10.0.3/glpi-10.0.3.tgz
```

Puis on le décompresse dans **/var/www/html**
```bash
tar -xvzf glpi-10.0.3.tgz -C /var/www/html
```

Ensuite, une fois que l’on aura téléchargé et décompressé ces derniers, **nous attribuons les droits** au serveur LAMP d’agir sur les fichiers, et nous pourrons enchaîner sur l’installation graphique. Pour cela, entrez la commande suivante.
```
chown -R www-data /var/www/html/glpi/
```

**Configuration de GLPI - SETUP**
Une fois l’installation en commande terminée, il faut désormais navigateur à l'adresse de notre machine virtuelle ```http://@IP_VM/glpi```

On doit tomber sur cette page.
![setup 0](assets/glpi0.png)
*Source*: OpenClassrooms

On accepte le contrat de licence, puis on arrive sur la configuration, puis on sélectionne **installation** dans la page qui suivante. GLPI vérifie que tous les prérequis à son fonctionnement sont présents.
![setup1](assets/glpi1.png)

Maintenant on connecte GLPI à notre base de données. La base de données et dans la même VM, donc on indique *localhost* comme adresse IP. On lui indique l'utilisateur et le mot de passe créer plus haut.
![setup1](https://user.oc-static.com/upload/2019/01/29/15487712420173_image28.png)

Si tout est OK, vous devriez voir la fenêtre suivante avec la base de données **glpi** apparaître. Vous devez la sélectionner pour la suite.
![setup3](https://user.oc-static.com/upload/2019/01/29/15487714238912_image12.png)

Puis nous attendons 5min environs pour que la base de données se crée. GLPI crée les tables et ajoute les données dans la base de données.
![fini db](https://user.oc-static.com/upload/2019/01/29/15487716699915_image9.png)

Quand l'installation est finie nous pouvons nous connecter avec **glpi** et mot de passe **glpi** .
![login](https://user.oc-static.com/upload/2019/01/29/15487717733595_image2.png)

**Configuration du proxy pour la marketplace**

**Configuration d'une clé pour accéder au marketplace**

#### Ajout de clients

Nous devons installer l'agent-glpi
```bash
apt install glpi-agent
```

Puis modifier le fichier */etc/glpi-agent/agent.cfg* et ajouter le paramètre:
```
server: http://192.168.250.2/glpi/
```

On démarrer le service *agent*
```
# Uniquement pour cette session
glpi-agent

# De façon permanente
systemctl enable glpi-agent
```

La commande nous retourne alors
```
[info] New Inventory from debian10...
```

### Centreon
#### Installation
Tout d'abord nous devons créer une machine virtuel centos7. Nous détaillerons pas le détail de son [installation](https://docs.centos.org/en-US/centos/install-guide/)

**Configuration réseau**
Pour un souci de praticité nous mettons la VM sur le réseau LAN, puis en fin d'installation remettons dans le réseau virtuel.

Création script setup_proxy.sh
```bash
export HTTP_PROXY=http://USER:PASSWORD@172.16.0.1:3128
export HTTPS_PROXY=https://USER:PASSWORD@172.16.0.1:3128
export http_proxy=http://USER:PASSWORD@172.16.0.1:3128
export https_proxy=http://USER:PASSWORD@172.16.0.1:3128
```

Ajout des variables de proxy
```bash
source setup_proxy.sh
```

Configuration de l'adresse IP
```bash
ip addr add @IP/16 dev ens192
ip route add default via 172.16.0.1
```

Configuration du fichier **resolv.conf** pour le DNS
```bash
nameserver 172.16.0.1
```

**Installation des paquets**

Mise à jour des dépôts
```bash
sudo yum check-update
```

Mise à jour de Centos
```bash
yum install -y centos-release-scl
```

Ajout du dépôt
```bash
yum install -y https://yum.centreon.com/standard/20.04/el7/stable/noarch/RPMS/centreon-release-20.04-1.el7.centos.noarch.rpm
```

Installation des paquets
```bash
yum install -y centreon centreon-database
```

On redémarre les services
```bash
systemctl daemon-reload
systemctl restart mariadb
```

On vérifie que les services se lancent au démarrage
```bash
systemctl enable rh-php72-php-fpm httpd24-httpd mariadb centreon cbd centengine gorgoned snmptrapd centreontrapd snmpd
```

Avant de se connecter on redémarre le serveur
```
systemctl start httpd24-httpd
```

**Configuration de la base de données**
On se connecte à l'invite de commande de MariaDB

Puis on crée la base de données et l'utilisateur

```bash
MariaDB [(none)]> create database centreon; 
MariaDB [(none)]> grant all privileges on centreon.* to glpi@localhost identified by "lannion"; 
MariaDB [(none)]> quit
```

**Setup de Centreon**
On se connecte depuis le navigateur à l'adresse IP de la machine virtuelle  ```http://@IP/centreon``` 
![welcome](https://docs.centreon.com/assets/images/acentreonwelcome-b07c2b25065f0495ce4c9cbfc8957c80.png)

Centreon vérifie les prérequis
![check](https://docs.centreon.com/assets/images/acentreoncheckmodules-25d6c00d349f180a1dfe1741a30692fa.png)

On laisse par défaut les répertoires
![directory](https://docs.centreon.com/assets/images/amonitoringengine2-02eec9e3f93c061a1d6c33b1c1731e0c.png)

Identique ici
![directory 2](https://docs.centreon.com/assets/images/abrokerinfo2-00c20b915921b5691f637397c6157e0c.png)

Configuration d'un administrateur
![admin account](https://docs.centreon.com/assets/images/aadmininfo-d162b145642bab07bf27f38f3da43696.png)

On donne les informations pour se connecter à la base de données
![base de données](https://docs.centreon.com/assets/images/adbinfo-97a9d2cf461739dd287a15491a626776.png)

On laisse centreon créer les différentes tables et se configurer
![done](https://docs.centreon.com/assets/images/aendinstall-4d2d941bd7964a581f863a6536b8a3c8.png)

On peu se connecter
![login](data:image/png;base64,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)


#### Ajout de clients
# Plan de reprises d'activitées
## ESXI: DCUI
### Sauvegarde DCUI
Avant même de sauvegarder, le service doit être activer depuis VCenter de l'ESXI.

On passe en mode maintenance de l'ESXI
```
vim-cmd hostsvc/maintenance_mode_enter
```

On synchronise la configuration actuelle de façon permanente
```
vim-cmd hostsvc/firmware/sync_config
```

On télécharge la configuration
```
vim-cmd hostsvc/firmware/backup_config
```

![backup](https://kb.vmware.com/servlet/rtaImage?eid=ka05G0000017FWt&feoid=00Nf400000Tyi5C&refid=0EM5G000005VnSP)

On optient un lien avec * qui doit être remplacer par l'addresse IP de l'ESXI ```http://@IP/downloads/123456/configBundle-xx.xx.xx.xx.tgz```

On sort du mode maintenance
```
vim-cmd hostsvc/maintenance_mode_exit
```

### Restauration DCUI

On passe en mode maintenance de l'ESXI
```
vim-cmd hostsvc/maintenance_mode_enter
````

Copiez le fichier de configuration de sauvegarde sur l'hôte ESXi ou sur un datastore disponible.

Redémarrez l'hôte. Cette étape est nécessaire, sinon le processus de restauration pourrait ne pas se terminer complètement.

Une fois que l'ESXi a redémarré, exécutez cette commande pour restaurer la configuration des hôtes ESXi (remplacez *repertoire* par le chemin vers le configBundle.tgz)

```
vim-cmd hostsvc/firmware/restore_config /<repertoire>/configBundle.tgz
```

## VMotion: Haute Disponibilité
### Configuration
- Dans la page d'accueil de vSphere Client, accédez à ``Accueil > Hôtes et clusters.``
- Cliquez avec le bouton droit sur le centre de données et sélectionnez ``Nouveau cluster``.
- Saisissez un nom pour le cluster.
- Sélectionnez les fonctionnalités de cluster vSphere HA.
### Ajout de clients dans le cluster
- Dans vSphere Client, accédez à un cluster d'un centre de données.
- Dans l'onglet Configurer, sélectionnez Configuration > Démarrage rapide.
- Cliquez sur Ajouter dans la carte Ajouter des hôtes.
- Sur la page Ajouter des hôtes, ajoutez des hôtes nouveaux ou existants au cluster.
    -   Ajoutez des hôtes qui ne font pas partie de l'inventaire de vCenter Server.
        1.  Cliquez sur l'onglet Nouveaux hôtes.
        2.  Renseignez les zones de texte Adresse IP et d'informations d'identification pour ces hôtes.
    -   Ajoutez des hôtes qui sont gérés par votre instance vCenter Server et qui sont dans le même centre de données que votre cluster.
        1.  Cliquez sur l'onglet Hôtes existants.
        2.  Dans la liste, sélectionnez les hôtes que vous souhaitez ajouter au cluster.
