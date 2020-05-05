---
Title: Mise en place de Foreman
Type: Doc
Nature: Notes
Création: 27/04/2020
---

# Mise en place de Foreman
****
## Table of Contents

- **[Introduction](#introduction)**
- **[Installation de Forman](#Installation-de-Forman)**
  - [Prérequis](#Prérequis)
  - [Téléchargement des dépôts yum](#Téléchargement-des-dépôts-yum)
  - [Installation de Forman Installer](#Installation-de-Forman-Installer)
- **[Intégration d'hôtes dans Foreman](#Intégration-d'hôtes-dans-Foreman)**
- **[Création d'un groupe d'hôtes](#Création-d'un-groupe-d'hôtes)**
- **[Installation des modules Puppet et plugings nécessaires](#Installation-des-modules-Puppet-et-plugings-nécessaires)**
  - [Installation du module ntp](#Installation-du-module-ntp)
  - [Installation du plugin foreman_remote_execution](#Installation-du-plugin-foreman-remote-execution)
- **[Sources](#Sources)**
****


## Introduction
[Forman](https://theforeman.org/introduction.html) est outil opensource dont le but est faciliter la gestion des serveurs tout au long de leur cycle vie, du provisionnement et de la configuration à l'orchestration et à la surveillance. L'[architecture](https://theforeman.org/manuals/2.0/index.html#3.2.2InstallerOptions) de Foreman est constituée de :
- une ``instance centrale de Foreman``, disposant d'une interface graphique, qui permet de configurer et de gérer les noeuds, les plugins.
- un [Smart Proxy](https://theforeman.org/manuals/2.0/index.html#4.3SmartProxies), qui, comme son nom l'indique, est un proxy et gère les services à distance (NTP, TFTP, DHCP, DNS, Puppet, Puppet CA, ...) sur les clients (noeuds) intégrés dans Foreman.
Pour étendre les fonctionnalités de Foreman, il existe de nombreux [plugins](https://theforeman.org/plugins/) dont  Ansible, Chef et Pupet (pour l'orchestration et la gestion des configurations), OpenSCAP (pour le scan et l'évaluation de conformité et de vulnérabilités), Azure (pour le cloud), Katelo et Docker (pour la gestion des conteneurs) ...
> Remarque: Il est fortement recommendé d'installer foreman sur une machine toute neuve pour éviter les problèmes, car l'outil apporte plusieurs modifications sur un système.

## Installation de Foreman
La plus simple manière [d'installer Forman](https://theforeman.org/manuals/2.0/quickstart_guide.html) est d'utiliser [Foreman installer](https://theforeman.org/manuals/2.0/index.html#3.2ForemanInstaller). C'est une collection modules Puppet qui installe tout le nécessaire pour configurer Foreman, Puppet master, et Smart Proxy par défaut. Cependant, ceci utilise Puppet (4 or later required) pour installer Foreman. Il est fortement recommandé d'installer l'outil sur une machine neuve car il apporte plusieurs modifications sur un système.
> Note : Pour les caractéristiques du système, voir [SystemRequirements](https://theforeman.org/manuals/2.0/index.html#3.1SystemRequirements)

Nous allons installer l'outil en mode standalone (même machine) sur une machine CentOS7 :
- IP  Address = 192.168.1.58
- Hostname = foreman.mydomaine.local
- SeLinux = Enabled
- Firewall = Enabled
- Pas de service DNS installé

Pour l'installation, il faut récupérer les dépôts yum nécesaires puis télécharger l'installer et le lancer.

### Prérequis
Avant l'installation, si on n'a pas de DNS, il faut configurer un nom de domaine (fqdn) pour la machine comme suite :
1. Editer le fichier `/etc/hosts` et ajouter cette ligne :
```
192.168.1.58  foreman.mydomaine.local
```
2. Renommer le nom de la machine :
```
sudo hostnamectl set-hostname foreman.mydomaine.local
```

### Téléchargement des dépôts yum
Avant l'installation, nous pouvons récupérer les différents dépôts nécessaires pour installer Foreman, à l'aide des commandes ci-dessous :
```
# Utilisation d'une version Puppet est recommandée
sudo yum -y install https://yum.puppet.com/puppet6-release-el-7.noarch.rpm
# Activer les dépôts EPEL, Foreman and SCL (Software Collections):
sudo yum -y install http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
sudo yum -y install https://yum.theforeman.org/releases/2.0/el7/x86_64/foreman-release.rpm
sudo yum -y install foreman-release-scl
```
### Installation de Forman Installer
Il est fortement recommendé d'installer Foreman via un un utitilitaire conçu pour ça, `Forman Installer`. Il peut être installé comme suit :
```
# Télécharger l'installer puis le lancer
sudo yum -y install foreman-installer
sudo foreman-installer
```
> Attention : Si vous n'utilisez pas une machine neuve, il se peut que la commande vous signale certains défauts de configurations sur le système. Il faut donc les corriger.

> Note :
> Par défaut, `foreman-installer` installe Foreman avec les paramètres par défaut. Cependant il peut être lancé avec des arguments  ou en mode interactif.
> - la syntaxe de la commande est `foreman-installer [OPTIONS]`, cf `foreman-installer --full-help | more`.
> - Tous les arguments passés à la commandés sont notés dans `/etc/foreman-installer/scenarios.d/foreman-answers.yaml`
> - Les logs de l'intaller se trouvent dans `/var/log/foreman-installer/foreman.log
`

Si l'installation est bien faite, vous verrez le message ci-dessous
>Preparing installation Done
>  Success!
>  * Foreman is running at https://foreman.mydomaine.local
>      Initial credentials are admin / 77PS7gwkdGK72g5K
>  * Foreman Proxy is running at https://foreman.mydomaine.local:8443
>  The full log is at /var/log/foreman-installer/foreman.log

> Pour information, les fichiers de configuration de Foreman se trouvent dans `/etc/foreman/`

Vous pouvez alors accéder à l'interface de Foreman via son URL, https://foreman.mydomaine.local ou https://IP_serveur, et vous connectez avec les crédentials par défaut fournis (`admin / 77PS7gwkdGK72g5K`). Cependant, s'il le firewall est activé, comme dans notre cas, il faut activer les ports nécessaires parmi la liste ci-dessous :

Port | Protocol  |  Required For
--|---|--
53  | TCP & UDP   |  DNS Server
67, 68  | UDP |  DHCP Server
69 |  UDP |   	* TFTP Server
80, 443  |  TCP |   	* HTTP & HTTPS access to Foreman web UI / provisioning templates - using Apache + Passenger
3000  |   	TCP |  HTTP access to Foreman web UI / provisioning templates - using standalone WEBrick service
5910 - 5930  | TCP  |  Server VNC Consoles
5432  | TCP  |  Separate PostgreSQL database
8140  | TCP  |  * Puppet Master
8443  |  TCP |  Smart Proxy, open only to Foreman

> Note :
> - `*` : indique les ports obligatoires
> - ces ports peuvent être activés comme suite :
> ```
  sudo firewall-cmd --permanent --add-port={67-69/udp,80/tcp,443/tcp,8140/tcp}
  sudo firewall-cmd --reload
  ```
> - Et n'oubliez pas de changer ces crédentails par défaut!

Et depuis l'interface, on peut voir la liste des machines en allant dans `Hotes > tous les hotes`. A cette étape de l'installation, on verra qu'une seule machine, celle sur laquelle Foreman est installé. Si ce n'est pas le cas, on peut utiliser la commande `sudo puppet agent --test` qui va installer l'agent puppet sur Foreman et dont ajouter automatiquement la machine dans la base de données de Foreman.

## Intégration d'un nouveau hôte dans Foreman
Pour intégrer un nouvel hôte dans Foreman suivez les étapes ci-dessous :
- **Sur l'interface de Forman**:  Création d'une signature automatique des certificats des hôtes
  1. Aller dans `Infrastructure > Smart Proxy`, sur le Proxy, développer la case `Modifier` puis sélectionner `Signature automatique`
  2. Dans `Puppet CA`, cliquer sur `Entrées de signature automatique`, puis sur `Créer une entrée de signature automatique` et créer le/les domaines de vos hôtes (ex: `*mydomaine.fr`).
- **Sur chaque hôte** : Installation de l'[agent Puppet](https://puppet.com/docs/puppet/latest/install_agents.html)
  1. Installer le paquet puppet-agent `sudo yum -y install https://yum.puppet.com/puppet6-release-el-7.noarch.rpm`
  2. Installer l'agent puppet `sudo yum install -y puppet-agent`
  3. Indiquer le nom du serveur puppet master (et éventuellement d'autres paramètres) dans le fichier de configuration de l'agent :
  ```
  [main]
  report = true
  [agent]
  server = foreman.mydomaine.local
  environment = production
  runinterval = 10m
  ```
  4. Démarrer l'agent en tant que service
  ```
   sudo /opt/puppetlabs/bin/puppet resource service puppet ensure=running enable=true
  ```
  5. Intégrer l'agent au serveur Foreman `sudo puppet agent --test`
  > Note :
  > - si `sudo puppet ...` ne fonctionne pas, spécifier le chemnin complet du binaire `sudo /opt/puppetlabs/bin/puppet`

Ensuite, on peut vérifier depuis l'interface de Forman que les hôtes sont intégrés. Pour cela :
  1. Retourner dans `Puppet CA > Certificats`, pour vérifier que les certificats sont bien signés.
  2. Aller dans `Hotes > Tous les hotes` pour voir l'hôte qui a été intégré.
> Note : Apparition des hôtes dans l'interface peut prendre quelques minutes.

## Création d'un groupe d'hôtes
Il est beaucoup plus simple de gérer les services et les configurations sur les hôtes à travers des groupes d'hôtes. POur créer un groupe d'hôtes dans Foreman :
1. Aller dans `Configurer > Groupe d'hôtes`
2. Renseigner les paramètres de configuration du groupe (nom, puppet master, classes puppets, réseau, système d'exploitation, environnements, ...)

Ensuite, il faut intégrer les hôtes dans le groupe. Pour cela :
1. Aller dans `Hôtes > Tous les hôtes`
2. Cocher la case de tous les hôtes concernés
3. Cliquer le bouton `choisir l'action` (en haut), puis sélectionner `Changer le groupe`
4. Dans le champs `Groupe d'hôtes`, sélectionner le groupe.

## Installation des modules Puppet et plugings nécessaires
Par défaut, Foreman s'appuie sur Puppet pour la configuration de Foreman et les hôtes (sur qui l'agent puppet est installé).
> Note :
> - vérifier d'abord que le serveur puppet tourne `sudo systemctl status puppetserver`
> - et que le port `8140` est bien en écoute `sudo netstat -lntup | grep 8140`

Après l'installation de Foreman, certains [modules Puppet](https://forge.puppet.com/modules) sont nécessaires pour le bon fonctionnement de l'outil, notamment le `module ntp`.

### Installation du module ntp
Ce module permet de gérer le service NTP pour les hôtes d'un environnement (par défaut, Production), important pour la synchronisation. Pour l'[installer](https://puppet.com/docs/puppet/latest/quick_start_ntp.html), on peut utiliser la commande ci-dessous :
```
sudo puppet module install puppetlabs/ntp
```
> Note :
> - si `sudo puppet ...` ne marche pas, utiliser `sudo /opt/puppetlabs/bin/puppet module install puppetlabs/ntp`
> - l'option `-i ou --target-dir` permet de spécifier le répertoire d'installation des modules (cf `puppet module install --help`)

Ensuite, il faut importer la `class ntp`. Cela est faisaible depuis l'interface de Foreman :
1. aller dans `configurer > classes`, cliquer sur `importer environnements de ...`
2. Si la classe ntp est bien installée, elle devrait figurer dans la liste des classes disponibles;
3. Cochez la case où figure ntp, pour l'environnement choisi (par défaut, Production), puis sur le bouton `update ou mise à jour`;
4. Cliquer sur la classe NTP sur la liste des classes importées;
5. Aller dans `Paramètres Smart Class` puis défiler jusqu'à `servers` et cliquer dessus pour configurer le NTP;
6. Cocher la case `Remplacer par`
7. sélectionner `tableau` pour `parameter type`
8. et dans le champs `valeurs par défaut`, entrer la liste des serveurs NTP (ex: `["0.fr.pool.ntp.org","1.fr.pool.ntp.org","2.fr.pool.ntp.org","3.fr.pool.ntp.org"]`)
9. Défiler jusqu'en bas et cliquer sur `Valider`
> Remarque :
> - Chaque module de puppet installé, sa classe doit être importée dans Foreman en suivant les étapes `1 à 3` pour le cas de ntp.
> - Sur la liste des serveurs ntp, on peut mettre ses serveurs ntp si on en a.

Ensuite il faut configurer les hôtes pour qu'ils utilisent le NTP. Pour ce faire :
1. Aller dans `Hôtes > Tous les hôtes`
2. Au niveau de chaque hôte, cliquer sur le bouton `Modifier`
3. Aller dans `Class puppet`
4. Cliquer sur la classe `ntp` et choisir le/les modules qui vous intéressent (ntp, ntp::config, ntp::install, ntp::service)
5. Cliquer sur valider
6. Attendre quelque minute et vérifier que le service ntp tourne :
```
sudo /opt/puppetlabs/bin/puppet resource service ntpd
```
> Note : La manière la plus simple d'associer une classe aux hôtes est de le faire avec un groupe d'hôtes. Pour cela, aller dans `Configurer > Groupe d'hôte` puis sélectionner le groupe et définir les classes puppet.

### Installation du plugin foreman_remote_execution
Le plugin [foreman_remote_execution](https://theforeman.org/plugins/foreman_remote_execution/1.7/index.html) est aussi indispensable pour exécuter des commandes arbitraires les hôtes depuis Foreman en utilisant divers providers (Ansible, Puppet, command, ...). Il peut être installé à l'aide de la commande ci-dessous :
```
 sudo foreman-installer --enable-foreman-plugin-remote-execution \
--enable-foreman-proxy-plugin-remote-execution-ssh
```
Pour tester le fonctionnement de ce plugin, nous allons réaliser un ping. Pour cela :
1. dans `Hôtes > Tous les hôtes`, sélectionner un hôte
2. dans `choisir l'action`, cliquer sur `Programmer une tâche à distance`
3. dans `catégorie du job`, sélectionner `commands`
4. dans `command`, mettre `ping -c 3 localhost`
> Note : on peut `Afficher les champs avancés` pour renseigner d'autres paramètres (utilisateur, mot de passe, ...)

5. cocher `Execute now` puis `valider` et voir le résultat.



## Sources
- Documentation
  - [Site officiel de Forman](https://theforeman.org/)
  - [Foreman Openscap](https://github.com/theforeman/foreman_openscap)
  - [Puppet Configuration]( https://puppet.com/docs/puppet/latest/configuration.html)
- Tutoriels :
  - [Foreman Quickstart Guide](https://theforeman.org/manuals/2.0/quickstart_guide.html)
  - [scaleway.com: foreman on ubuntu](https://www.scaleway.com/en/docs/how-to-install-and-configure-foreman-on-ubuntu-xenial/)
  - [linuxtechi.com: Foreman on centos 7](https://www.linuxtechi.com/install-and-configure-foreman-on-centos-7-x/)
  - [itzgeek.com: foreman-on-centos-7-rhel-7-ubuntu](https://www.itzgeek.com/how-tos/linux/centos-how-tos/install-foreman-on-centos-7-rhel-7-ubuntu-14-04-3.html)
