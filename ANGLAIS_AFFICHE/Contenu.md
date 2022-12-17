[padlet](https://padlet.com/annepullen/ipykej6ls9lzhicd)
# Introduction to containerisation
## C'est quoi la containerisation ?
- [ ] Baser sur LinuX Containers: Méthode de cloisonnement  de l'OS. Le principe est de faire tourner des environnements de linux isoler des un des autres par isolation conteneur, partageant un même noyaux. Par rapport à une machine virtuelle, il ne possède pas d'OS, mais sur les fonctionnalités du noyaux de l'OS hôte.
**Schema 0**
![lxc](./img/lxc.png)
Le conteneur virtualise l'environnement d'exécution (CPU, RAM, File system) 
**Schéma A**
![lxc2](./img/lxc1.png)

LCX se repose sur deux groupes:
- Cgroups = Control groups => Isoler et limiter les ressources
- Namespace = Cloisonnement des espaces de nommage. Empecher un groupe de voir les ressources d'un autre groupe.
**Schéma B**
![lxc2](./img/lxc2.png)

La virtualisation par conteneur est caractérisé par la couche "contrôleur". Il gère l'ensemble des fonctionnalités pour les conteneurs. L'interraction entre le conteneur est l'OS, gestion de la sécurité par les des privilèges et des ressources, la scalabilité est duplication des conteneurs. L'accès au conteneur par des API et CLI, portabilité, migration à chaud ou à froid du conteneur

## VM vs Containerisation
Une VM recrée totalement un serveur, avec un OS complet, ces pilotes, ces fichiers binaires et librairies et l'application elle même. Vue le container n'embarque pas d'OS, il est beaucoup plus léger que la VM. Ainsi il est plus simple à migrer, télécharger, sauvegarder et restaurer. Il permet à l'hôte d'héberger plus de containers que de VM.

Docker est une technologie de containerisation qui se repose par LXC de Linux. Docker va permettre de créer des conteneurs contenant les dépendances nécessaires à l'execution d'une application. Il permet son execution dans un isolement virtualisé. Docker Engine le rôle de controler docker.

Son architecture: Client - Serveur.
Le client communique avec le Docker Daemon, qui fait tourner le docker Engine. Le client peut tourner sur la même machine du daemon ou sur une autre machine.
![docker](./img/docker1.png)
# Image 
![img](./img/docker2.png)
L'environnement d'exécution d'un container nécessite l'empilement d'image. L'image docker est un template prête à l'emploi, prêt à la création d'un conteneur . Une image ne peut être modifier directement, elle reste en lecture seule. La derniere couche contiendra toute les modifications nécessaires à l'application. L'image peut être créer par docker file, ou une image existante, ou depuis un registre d'image (Docker Hub)
![docker3](./img/docker3.png)

# Pourquoi populaire ?
Il permet d'accélérer et de faciliter les déploiements. Le conteneur contient toute les dépendances de son application, il est indépendant de l'infrastructure de son hôte. L'admin système à juste besoin de déployer le conteneur. Il n'a plus besoin de gérer la version d'une librairie, ou d'une application, sur l'ensemble des serveurs. 

# Introduction to containerisation
# The benefits
# Containerisation and cloud providers