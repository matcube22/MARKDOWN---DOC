[padlet](https://padlet.com/annepullen/ipykej6ls9lzhicd)
# Introduction to containerisation
## C'est quoi la containerisation ?
- [ ] Baser sur LinuX Containers: Méthode de cloisonnement  de l'OS. Le principe est de faire tourner des environnements de linux isoler des un des autres par isolation conteneur, partageant un même noyaux. Par rapport à une machine virtuelle, il ne possède pas d'OS, mais sur les fonctionnalités du noyaux de l'OS hôte.
**Schema 0**
![lxc](./img/lxc.png)
Le conteneur virtualise l'environnement d'exécution (CPU, RAM, File system) 
**Schéma A**
![lxc2](./img/lxc2.png)

LCX se repose sur deux groupes:
- Cgroups = Control groups => Isoler et limiter les ressources
- Namespace = Cloisonnement des espaces de nommage. Empecher un groupe de voir les ressources d'un autre groupe.
**Schéma B**
![lxc1](./img/lxc1.png)

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
## Waht is containerization
- [ ] based on LinuX Containers : OS partitioning method. The principle is to run linux environments isolated from eash other by container insulation, sharing the same core. Compert to the virtualization, it does not have a OS, but on the features of the host OS kernel.
**Schéma 0**
![lxc](./img/lxc.png)
The container virtualizes the execution environment (CPU, RAM, File system) 
**Schéma A**
![lxc2](./img/lxc2.png)
LCX consist of 2 groups:
- Cgroups = Control Groupe => Isolate and limited the use of ressources
- Namespace = Partitioning of name spaces. To avoid a group from seeing ressources to another groupe
- **Schéma B**
![lxc1](./img/lxc1.png)
Container-based virtualization is characterized by the layer "controller". It manage the whole of functionality for the container, the interaction between containers and the host OS, management of the security by privilege and resources, scalability and duplication of containers. Access to the container through APIs and CLI, portability, hot or cold migration of the container.
# The benefits
## Contenarization VS Virtual Machine
Virtual Machine fully recreates a server, with a full-featered OS, they pilots, they binary file, libary and the application itself. In contrast, the Contenarization not have a OS, it's more light than the VM. So, it's more easier to migrate, download, save and resore. It's allows to the host to have more containerisation than VM, the developer can pass more time on its application.
Docker is a technology of containerisation witch relies on LCX from Linux. Docker will allows to create containers containing the necessary dependencies for the execution of the application. It allows the execution in virtualized isolation. Docker engine is the controler of Docker container.
Its architecture : Client - Server.
The client communicated with the Docker Daemon, witch execute the Docker Engine. The client is on the same PC as the daemon or on the another PC.
![docker](./img/docker1.png)
## Docker image 
![img](./img/docker2.png)
The execution environment of container requires image stacking. The Docker image is a ready-to-use template, ready to create a container. The image can't be edit directly, it remains in read-only mode. The last layer will containe all necessary modifications for the application. The image can be create by a Docker file, an existing image or from a register image (Docker Hub).
## What is Docker is popular
It allows for faster and easier deployments. The container contains all the dependence  of the application, it's independent to the IT infrastructure of the host. The system administrator just needs to deploy the container. He no longer needs to manage the version of library, application, on all the server.
# Containerisation and cloud providers
Docker is one on the most use for containerization, but outher platform exists:
- Kubernetes : it's create by google but it was donated to Cloud Native Computing Foundation
### Source
For Docker : https://www.youtube.com/watch?v=caXHwYC3tq8
