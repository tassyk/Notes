---
Title: Installation du logiciel Variation sous centos
Version:
Date: 06/02/2020
Auteur: TK
---


# Introduction
La solution [Variation](https://www.variation.fr/) a été développée en coopération avec plusieurs établissements du social et médico-social (ESSMS) afin de répondre à leurs besoins en matière de centralisation des dossiers d'usagers et d'échange d'information entre intervenants. Avec Variation, vous renforcez la qualité de votre gestion collective et affinez la prise en charge individuelle de vos usagers.
Le logiciel Variation se présente sous la forme d'une application web.  Elle doit être installée sur un serveur Linux hébergeant les services Apache2, PostgreSQL et PHP5 ou supérieur.

# Installation du logiciel Variation sous centos
**Remarque :** Comme les serveurs de démonstrations de l'éditeur tournent actuellement sur une version Ubuntu 18.04 LTS, PHP 7.2 et PostgreSQL 10.6, nous allons rester dans cet environ (sauf Ubuntu 18.04 LTS qui est remplacé par centos 7).
Les procédures d'installation sont expliquées [ici](https://dev.variation.fr/installation.php)

## Installation et configuration d'Apache et de PHP 7.2
###  Installer Apache
Apache (httpd) s'installe via la commande
```
sudo yum install -y httpd
sudo yum install -y httpd-tools yum-utils
```
**NB :** httpd-tools est une collection de scripts d'administration et de sécurité du serveur apache sous centos.

### Installer PHP 7
Pour installer PHP 7, procéder comme suite:
1. Récupérer le dépôt rémi pour qui contient les paquets php
```
sudo yum install -y wget https://rpms.remirepo.net/enterprise/remi-release-7.rpm
```
2. Activer php 7 à l'aide de la commande
```
sudo yum-config-manager --enable remi-php74
```
3. Installer php avec la commande:
```
# Verifier si le paquet php7.4 existe
sudo yum provides php
# Installer le paquet
sudo yum install -y php php-common
# Vérifier que php 7.4 s'est correctement installé
sudo rpm -q php
```
Installer aussi php-pgsql. C'est un module de php pour interagir avec la base de données PostgreSQL.
```
sudo yum install -y php-pgsql
```
Installer les modules php-xml et php-mbstring (Ces modules m'ont permis de corriger les erreurs par rapport à l'arragement lors de l'installation)
```
sudo yum install -y php-mbstring php-xml
```

## Configurer Apache et PHP
Nous allons autoriser l'utilsation des fichiers **.htaccess** dans Apache. Ces fichiers permettent de protéger un répertoire donné ainsi que ces contenus. Pour cela:
1. Activer d'abord le module **rewrite (mod_rewrite)** d'apache en ajoutant (s'il n'existe pas) la ligne **LodModule** dans le fichier **/etc/httpd/conf.modules.d/00-base.conf**
```
# cat /etc/httpd/conf.modules.d/00-base.conf
LoadModule rewrite_module modules/mod_rewrite.so
```
2. Ajouter/décommenter ces lignes dans le fichier de configuration d'apache, pour autoriser **.htaccess**
```
#sudo vi sudo vi /etc/httpd/conf.d/00-default.conf
<VirtualHost *:80>
  ServerAdmin email@domain.com
  DocumentRoot "/var/www/html"

   <Directory /var/www/variation/html>
        AllowOverride All
   </Directory>
</VirtualHost>
```
3. Redémarrer le serveur httpd pour prendre en charge les modifications
```
sudo systemctl restart httpd
```
Pour PHP, une configuration minimale est requise pour l'application variation.
- Adapeter ces lignes dans le fichier **/etc/php.ini**
```
...
memory_limit = 512M
post_max_size = 200M
upload_max_filesize = 100M
max_file_uploads = 50
...
```

**A Recommencer**
# Installation et configuration de la base de données PostgreSQL
Pour installer la base de données PostgreSQL, nous allons procéder comme suite:
1. Installer **postgresql postgresql-contrib** avec la commande suivante:
```
sudo yum install -y postgresql-server postgresql postgresql-contrib
```
> Attention : les paquets peuvent provenir des dépôts `base ou updates` de centos. Si c'est le cas, désactiver-les et réinstaller les paquets

- **/var/lib/pgsql/10/data/pg_hba.conf** est le fichier de configuration
- **/var/lib/pgsql/initdb.log** est le fichier de log après Initialisation de la base.


2. Initialiser et démarrer la base de données
```
sudo /usr/pgsql-10/bin/postgresql-10-setup initdb
```
3. Editer le fichier **/var/lib/pgsql/10/data/pg_hba.conf** et remplacer **indent** par **md5** comme suite pour autoriser la connexion à la base de données par un mot de passe.
```
...
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5
...
```
4. Editer le fichier **sudo vi /var/lib/pgsql/10/data/postgresql.conf** et definir les adresses d'écoute:
```
# Ecouter sur des ip spécifiques
listen_addresses = 'localhost, my_ip'
# ou ecouter sur toutes les ip
listen_addresses = '*'  
```
5. Démarrer le service postgresql
```
sudo systemctl start postgresql-10
sudo systemctl enable postgresql-10
```
6. Créer un utilisateur de la base de données de l'application
```
sudo su postgres
postgres$ psql
postgres=# create user variation password 'unmotdepasse';
postgres=# create database variation with encoding='UTF8' owner=variation;
postgres=# \q
```
7. Installer l'extension **pgcrypto** sur la base de données variation que vous venez de créer, extension nécessaire à l'application Variation
```
postgres$ psql variation
postgres=# CREATE EXTENSION pgcrypto WITH SCHEMA public;
postgres=# \q
```
8. Revenir à l'utilisateur initial et faire un test de connexion. Le mot de passe que vous avez donné précédemment vous sera demandé :
```
postgres$ exit
util$ psql -h localhost -U variation variation
postgres=# \q
```

## Installation du programme Variation
Pour installer le programme variation, procéder comme suite:
1. Créez un nouvel utilisateur système variation, qui sera utilisé pour l'installation et la mise à jour de l'application.
```
sudo adduser variation
sudo passwd variation
```
2. Une fois créé, donnez les droits à ce nouvel utilisateur pour le répertoire d'installation (/var/www/html).
```
sudo chown variation:variation /var/www/html
```
3. Puis connectez-vous avec ce nouveau compte et créez un répertoire (install) qui contiendra les fichiers d'installation et téléchargez les fichiers d'installation.
```
sudo su - variation
variation$ mkdir install
variation$ cd install
variation$ wget https://adullact.net/frs/download.php/file/8275/install.sh
```
4. Editez le script install.sh afin d'ajuster les paramètres suivants:
- **RELEASE** : Release à installer depuis le dépôt git Adullact. Il est conseillé de laisser la valeur par défaut.
- **VARIATION_PASSWD** : Mot de passe du compte administrateur de variation qui permet principalement de créer d'autres comptes utilisateurs. Le login par defaut pour ce compte est variation et le mot de passe associé doit être renseigné dans cette variable.
- **DESTDIR** : Répertoire d'installation de l'application (par défaut /var/www/html)
- **PGHOST** : Machine sur laquelle sera installée la base de données de l'application, localhost si vous installez la base de données sur la même machine que les sources du programme.
- **PGBASE** : Nom de la base de données.
- **PGUSER** : Utilisateur permettant d'accéder à la base.
- **PGPASS** : Mot de passe de l'utilisateur précédent.
- **ARRANGEMENT** : Répertoire dans lequel est placé l'arrangement à installer. La valeur **. (point)** indique d'installer l'arrangement par défaut fourni avec le logiciel. Pour utiliser un autre arrangement, décompressez le fichier d'arrangement dans un répertoire particulier et indiquez ce répertoire (qui doit contenir un répertoire "arrangement").
- **GIT** : Mettre cette variable à true pour installer un clone du dépôt git Adullact, plutôt qu'une release. à utiliser notamment pour travailler sur les sources du projet.
5. Exécutez le script d'installation
```
sh install.sh
```
**NB** : Vérifier le fichier de log `install-date.log` pour voir s'il n'y pas d'erreurs lors de l'installation

## Désinstallation du programme en cas d'erreur
En cas d'erreur lors de l'installation, il faut désinstaller le programme et le réinstaller. Pour cela :
1. Supprimer la base de données et la recréer
```
util$ sudo su postgres
postgres$ psql
postgres=# drop database variation;
postgres=# create database variation with encoding='UTF8' owner=variation;
postgres=# \q
postgres$ psql variation
postgres=# CREATE EXTENSION pgcrypto WITH SCHEMA public;
postgres=# \q
```
2. Supprimer le contenu du répertoire web `/var/www/html`
3. Corriger les erreurs PHP si nécessaire (en réinstaller les paquets manquants, par exemple)
4. Réexécuter le script d'installation
5. Redémarrer le service httpd et postgresql

## Tâches périodiques
Certaines tâches doivent être lancées quotidiennement en début de journée pour le bon fonctionnement de l'application. Pour cela :
1. Ouvrez le crontab de l'utilisateur `variation`
```
crontab -e
```
2. Ajouter cette ligne
```
00 6 * * * /var/www/html/cron.php jour
```

## Sauvegarde des données
Plusieurs types de données sont à sauvegarder : les bases de données, les photos des usagers et les documents affectés aux usagers.
1. Sauvegarder la base de données en créant un export

```
variation$ cd $HOME
variation$ mkdir sauvegardes
variation$ crontab -e

# dans crontab
00 23 * * * /usr/bin/pg_dump variation > /home/variation/sauvegardes/variation-bdd.sql
```

2. Sauvegarder les fichiers et photos des répertoires `photos/` et `doc_fichiers/` du répertoire web d'installation.

## Accès à l'application
L'accès pour l'utilisateur se fait à l'URL : http://votre-serveur/

Deux logins sont disponibles dès l'installation :
- le login variation associé au mot de passe précisé dans le script d'installation,
- le login admin associé au mot de passe admin qu'il vous faudra modifier dès que possible.

L'accès à la partie administration se fait à l'URL : http://votre-serveur/admin.php

## Quelques notes sur la commande psql
Voici quelques commandes utiles de psql pour manipuler une base de données :
- Se connceter à la base de données `variation` en tant que utilisateur `varaiation`:
```
psql -h localhost -U variation variation
```
- Lister les bases de données
```
variation=> \list
```
- Se connceter à la base de données variation
```
variation-> \c variation;
```
- Lister les tables de variation
```
\dt
```

## Liens
- Programme Variation:
  - produits : https://www.variation.fr/
  - Installation : https://dev.variation.fr/installation.php
  - Procédures de mise à jour du programme: https://dev.variation.fr/maj.php
  - depôt Github: https://github.com/actimeo/variation
  - demo : https://dev.variation.fr/documents/tutoriel.pdf
  - Erreur sur arrangement: https://adullact.net/forum/forum.php?set=custom&forum_id=3072&style=nested&max_rows=25
- Apache, PHP, PostgreSQL:
  - Tutos apache: https://www.microlinux.fr/apache-centos-7/
  - Tutoss Httpd mod rewrite: https://devops.ionos.com/tutorials/install-and-configure-mod_rewrite-for-apache-on-centos-7/
  - Tutos postgresql:
    - https://www.hostinger.com/tutorials/how-to-install-postgresql-on-centos-7/
    - https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-centos-7
    - https://chartio.com/resources/tutorials/how-to-list-databases-and-tables-in-postgresql-using-psql/
    - [Upgrade postgresql](https://fojta.wordpress.com/2019/02/11/upgrade-postgresql-version-9-to-10/)
  - Security postgresql: https://severalnines.com/database-blog/how-secure-your-postgresql-database-10-tips
  - Tutos PHP 7: https://linuxize.com/post/install-php-7-on-centos-7/
