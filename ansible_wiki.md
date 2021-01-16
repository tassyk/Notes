Utilisation d'Ansible
=====================
<table id="toc" class="toc"><tbody><tr><td><div id="toctitle">

## Sommaire
*   [<span class="tocnumber">1</span> <span class="toctext">Introduction</span>](#Introduction)
*   [<span class="tocnumber">2</span> <span class="toctext">Préambule</span>](#Préambule)
*   [<span class="tocnumber">3</span> <span class="toctext">Installation d'Ansible</span>](#Installation d'Ansible)
*   [<span class="tocnumber">4</span> <span class="toctext">Inventaire des serveurs</span>](#Inventaire_des_serveurs)
*   [<span class="tocnumber">5</span> <span class="toctext">Exécution d'actions sur les machines</span>](#Exécution_d'actions_sur_les_machines)
    *   [<span class="tocnumber">5.1</span> <span class="toctext">Ansible en ligne de commande</span>](#Ansible_en_ligne_de_commande)
    *   [<span class="tocnumber">5.2</span> <span class="toctext">Scripts Ansible: les Playbooks</span>](#Scripts_Ansible_:_les_Playbooks)
        *   [<span class="tocnumber">5.2.1</span> <span class="toctext">Structure d'un playbook</span>](#Structure_d'un_playbook)
        *   [<span class="tocnumber">5.2.2</span> <span class="toctext">Exécution d'un playbook</span>](#Exécution_d'un_playbook)
    *   [<span class="tocnumber">5.3</span> <span class="toctext">Roles Ansible</span>](#Roles_Ansible)
    *   [<span class="tocnumber">5.4</span> <span class="toctext">Création d'un rôle</span>](#Création_d'un_rôle)
    *   [<span class="tocnumber">5.5</span> <span class="toctext">Exemple de rôle</span>](#Exemple_de_rôle)
    *   [<span class="tocnumber">5.6</span> <span class="toctext">Exécution du rôle</span>](#Exécution_du_rôle)
*   [<span class="tocnumber">6</span> <span class="toctext">Variables</span>](#Variables)
    *   [<span class="tocnumber">6.1</span> <span class="toctext">Types de variables</span>](#Types_de_variables)
    *   [<span class="tocnumber">6.2</span> <span class="toctext">Utilisation des variables</span>](#Utilisation_des_variables)
*   [<span class="tocnumber">7</span> <span class="toctext">Templates</span>](#Templates)
    *   [<span class="tocnumber">7.1</span> <span class="toctext">Quelques syntaxes jinja</span>](#Quelques_syntaxes_jinja)
    *   [<span class="tocnumber">7.2</span> <span class="toctext">Exemples de templates</span>](#Exemples_de_templates)
    *   [<span class="tocnumber">7.3</span> <span class="toctext">Utilisation des templates</span>](#Utilisation_des_templates)

</td></tr></tbody></table>

Introduction
------------

Ansible est un outil libre d'automatisation et de gestion des
configurations. Il permet d'automatiser les tâches d'administration sur
les serveurs comme:

-   la configuration
-   le déploiement des applications
-   l'orchestration

L'objectif de cet article est de vous donner les bases afin de
comprendre le fonctionnement de l'outil.
 Pour plus d'information sur l'outil, voir la [documentation officielle](https://docs.ansible.com/ "https://docs.ansible.com").

Préambule
---------
Ansible est un outil développé en ``python``. Il s'appuie sur le protocole
``SSH`` pour l'accès à distance des serveurs.
 L'administration des serveurs via Ansible peut se faire:
-   en ligne de commande grâce à la commande ``ansible``
-   à l'aide d'un script Ansible: ``PLAYBOOKS``. Ces derniers sont
    écrits sous le langage ``YAML``
-   à l'aide d'un ``rôle`` Ansible

Les machines à administrer sont renseignées dans un fichier
d'inventaire. D'autres notions sont également importantes à savoir sur Ansible: les ``variables`` et les ``templates``. Chacun de ces concepts sera expliqué plus en détails dans les paragraphes qui suivent.

Installation d'Ansible
----------------------
Sous CentOS ou Redhat, Ansible peut être obtenu grâce à la commande
ci-dessous:
```
$ sudo yum install ansible
```
Cette commande crée le répertoire ``/etc/ansible`` dans lequel on trouve
les fichiers de configuration (``ansible.cfg``) et d'inventaire
(`hosts`).
 Par défaut, Ansible utilise la clé SSH de l'utilisateur pour se
connecter sur les machines. Une fois Ansible est correctement installé,
il faut donc ajouter cette clé sur le fichier `authorized_keys` si l'on souhaite utiliser cette méthode de connexion. Sinon, il faudra ajouter une option (`-k`) sur les commandes ansible pour utiliser l'authentification via un mot de passe.

Inventaire des serveurs
-----------------------

Avant d'exécuter des actions sur les machines via Ansible, il faut,
avant tout, les renseigner dans un fichier d'inventaire.
Par défaut, ce fichier se nomme **hosts**. Il est possible d'utiliser
un fichier différent. Mais dans ce cas, il faut préciser le fichier en
question dans les commandes ansible avec l'option (**-i**). Dans un fichier
d'inventaire, on peut déclarer les machines de différentes manières:

-   Individuellement (par son nom ou IP)
-   Par groupe de machines
-   Par groupe de groupe de machines

**Exemple :**
```
    $ sudo cat /etc/ansible/hosts

    #individuels
    blue.example.com
    192.168.100.1
       #hosts avec des variables
    host1 http_port=80 maxRequestsPerChild=808
    host2 http_port=303 maxRequestsPerChild=909

    #groupe de machines nommé webservers
    [webservers] #nom du groupe entre crochets
    alpha.example.org
    192.168.1.110

    #groupe de machines nommé dbservers
    [dbservers] #nom du groupe entre crochets
    beta.example.org
    192.168.1.110

    #groupe de groupe de machines nommé allservers
    [allservers:children] #groupe de groupe indiqué par le mot clé: children
    dbservers #groupe
    webservers #groupe
```

Comme le montre ce fichier, les machines peuvent être déclarées en
précisant soit leur IP ou leur nom.\
 Le nom d'un groupe de machines est mis **entre crochets**. Sous ce nom, on
déclare les machines du groupe. Le mot clé **children** permet de créer un groupe de groupes de machines.\
 Après avoir constitué un fichier d'inventaire, on peut exécuter
maintenant des actions sur les machines.


Exécution d'actions sur les machines
------------------------------------

On peut effectuer des tâches sur les machines définies dans le fichier
d'inventaire soit:
-   en ligne de commande;
-   via un playbook;
-   via un rôle.


### Ansible en ligne de commande

Chaque action est une fonction. Sous ansible, une fonction se désigne par
 **modules**. Il existe une pléthore de modules. Par exemple, on a les modules **ping, yum, files, copy, systemd**, ...\
 Les tâches en ligne de commande sont lancées via la commande:
**ansible**. Voici la syntaxe générale :
```
    ansible <host-pattern> [options]
```
Les options sont disponibles sur cette [adresse](http://docs.ansible.com/ansible/latest/cli/ansible.html\#synopsis).
On peut en citer:

**-k, --ask-pass** : prompt pour une demande de mot de passe\
 **-m \<MODULE\_NAME\>, --module-name \<MODULE\_NAME\>** : pour spécifier le
nom du module \
 **-i, --inventory, --inventory-file** : pour spécifier un fichier
d'inventaire (différent de hosts).

Voici quelques exemples de tâches qu'on peut exécuter en ligne de
commande:
```
    # test de connectivité sur toutes les machines (all)
    $ ansible all -m ping -a

    # copie de fichier sur les machines du groupe webservers
    $ ansible webservers -m copy -a "src=/etc/hosts dest=/tmp/hosts"

    # création de fichier "foo" sur les machines dbservers avec permission 600
    $ ansible dbservers -m file -a "dest=/srv/foo/a.txt mode=600"

    # installation d'apache sur les machines webservers
    $ ansible webservers -m yum -a "name=httpd state=present"
```
**Notes**: la liste des module est disponible sur la page [ All modules](https://docs.ansible.com/ansible/2.8/modules/list_of_all_modules.html).


### Scripts Ansible: les Playbooks

Un script Ansible se désigne sous le nom de **Playbook**. Il contient les modules et permet de définir un ensemble de tâches à exécuter sur les machines.

#### Structure d'un playbook

La structure d'un playbook ressemble à ceci :
```
    ---
    - hosts: webservers
      vars:
        http_port: 80
        max_clients: 200
      remote_user: root
      tasks:
      - name: ensure apache is at the latest version
        yum:
          name: httpd
          state: latest
      - name: write the apache config file
        template:
          src: /srv/httpd.j2
          dest: /etc/httpd.conf
        notify:
        - restart apache
      - name: ensure apache is running (and enable it at boot)
        service:
          name: httpd
          state: started
          enabled: yes
      handlers:
        - name: restart apache
          service:
            name: httpd
            state: restarted
```

On voit que le playbook contient plusieurs sections :

-   **hosts** : permet de déclarer les hosts ou groups (ici webservers)
    sur lesquels le playbook doit se jouer **(all = toutes les
    machines)**.
-   **vars** : ici on déclare nos variables. On traitera les variables
    plus tard.
-   **tasks** : les différentes tâches que notre playbook va devoir
    exécuter :
    -   Module **yum**: installation de la dernière version d'apache.
    -   Module template**: copie du fichier de configuration apache avec
        prise en compte des variables. On reviendra sur les templates.
    -   Module **notify**: notifier une tâche donnée(ici: template).
        Cela permet d'exécuter des traitements conditionnels par la
        suite en cas de besoin.
    -   Module **service**: démarrage et activation au boot du du daemon
        httpd.
    -   **handlers**: C'est une tâche particulière. Il sert à effectuer
        un traitement spécifique (ici démarrage du service httpd ) quand
        toutes les tâches sont correctement exécutées.

**Remarques :**
 - Chaque debut de playbook commence par **---**
 - Le module hosts est toujours précédé d'un **tiret**
 - On voit que dans la partie **"tasks"**, chaque tâche est précédée du
module **" - name"** (ne pas oublier le **tiret** devant!). Il permet de
préciser un nom pour la tâche.
 Cela permet de savoir qu'elle tâche est entrain d'être exécutée lors de
l'exécution du playbook.
 - Définir un nom pour une tâche n'est pas obligatoire (pas obligatoire
d'indiquer "- name "). Cependant, dans le cas échéant, cette tâche doit
être précédée d'un **tiret**
 Exemple:
```
    - service:
          name: httpd
          state: started
          enabled: yes
```

- Les arguments d'un module peuvent s'écrire de deux manières:
  - Sur la même ligne que le module en utilisant la syntaxe **key=value**
```
      service: name=httpd state=started enabled=yes
```
  - Ou en dessous du nom du module (comme vu plus haut) en utilisant la
syntaxe **key: value**
```
      service:
          name: httpd
          state: started
          enabled: yes
```
- Il faut respecter l'indentation:

  - Toutes les parties (vars, tasks, handlers) sont situés au même niveau
par rapport à **hosts**.

  - Tous les modules (yum, service, ...) de la partie **tasks** sont situés
au même niveau. Pour chaque module, tous les arguments sont situés
également au même niveau.

#### Exécution d'un playbook

On exécute un playbook Ansible via la commande **ansible-playbook**.\
 **Syntaxe:** `ansible-playbook nom_playbook.yml [options]` \
 Parmi les options, on peut citer :
-   **-i** ou **--inventory=INVENTORY_FILE** : spécifier le nom du
    fichier d'inventaire. Pour le fichier par défaut, hosts, cette
    option est facultative.
-   **-u** ou **--user=USER** : pour spécifier le nom de l'utilisateur qui
    exécute le playbook.
-   **-k** ou **--ask-pass** : pour demander le mot de passe de l'utilisateur
    pour se connecter via SSH.
-   **--ask-vault-pass**: avec ansible-vault, il est possible de créer
    un fichier contenant des secrets. Dans ce cas, on utilise cette
    option pour demander ces secrets.
-   **-l** ou **--limit=MACHINES**: pour limiter l'exécution du playbook à
    quelques machines.
-   **-t** ou **--tags=TAG**: exécuter certaines tâches uniquement
    identifiées par un tag.

**Exemples d'exécution :**
```
    #sans option suppose que le fichier d'inventaire est hosts, user est l'utilisateur courant, et accès aux machines se fait via clé ssh
    $ ansible-playbook proxy.yml

    #fichier d'inventaire est différent: inventory/hosts_qualif
    $ ansible-playbook proxy.yml -i inventory/hosts_qualif

    #accès SSH se fait via mot de passe
    $ ansible-playbook proxy.yml -i inventory/hosts_qualif -k

    #accès SSH se fait via mot de passe et user=root
    $ ansible-playbook proxy.yml -i inventory/hosts_qualif -k -u root

    #limiter aux groupes webservers
    $ ansible-playbook proxy.yml -i inventory/hosts_qualif -k -u root -l webservers

    #exécuter uniquement les tâches identifiées par le tag nginx <br>
    $ ansible-playbook proxy.yml -i inventory/hosts_qualif -k -u root -t nginx
```

### Roles Ansible

On peut faire beaucoup de choses avec les playbooks et les includes.
Cependant, lorsque nous commençons à gérer des infrastructures
complexes, on a beaucoup de tâches à exécuter et dans ce cas, les rôles s’avèrent
salvateurs dans l’organisation et l’abstraction qu’ils apportent.

#### Création d'un rôle

Ansible met à disposition une plate-forme permettant de télécharger et
de partager des rôles avec la communauté des utilisateurs : Ansible
galaxy.Ce qui est un bon moyen de ne pas réinventer la roue.\
 Avec les rôles, nos playbooks seront à la **racine de notre répertoire
Ansible** (ou dans un dossier que l'on nommera *playbooks*) tandis que
les rôles seront dans **roles**.
```
    $ cd /etc/ansible && ls

    hosts           roles       playbooks           ansible.cfg            README.md
    rsyslog.yml     proxy.yml       testplaybook.yml
```
Ainsi, lorsque nous appellerons un rôle depuis un playbook, sans avoir
besoin de préciser autre chose que son nom, Ansible saura où chercher!\
 En CLI, le moyen le plus rapide de créer toute l'arborescence d'un rôle
vide, est l'utilisation de la commande **ansible-galaxy**.\
 Dans le dossier **roles**, initialisons un role vide appelé
***webservices*** à l'aide de la commande ci-dessous:
```
    $ ansible-galaxy init webservices
```
Cette commande va créer le rôle **webservices** avec toute son
arborescence :
```
    cd webservices && ls -R
    README.md   handlers    tasks       vars
    defaults    meta        tests

    ./defaults:
    main.yml

    ./handlers:
    main.yml

    ./meta:
    main.yml

    ./tasks:
    main.yml

    ./tests:
    inventory   test.yml

    ./vars:
    main.yml
```
On voit ici qu’Ansible nous a créé plusieurs dossiers avec un fichier
**main.yml** dans chacun d’eux. Par défaut, Ansible ira chercher les
informations dans les main.yml de chaque répertoire. Cependant, vous
pouvez créer d’autres fichiers **.yml** et les référencer dans vos
instructions (main.yml) à l'aide des includes.

-   **defaults**: Ce sont ici les variables par défaut qui seront à
    disposition du rôle.
-   **vars**: De la même manière que defaults, il s’agit ici de
    variables qui seront à disposition du rôle, cependant, celles-ci ont
    en général vocation à être modifiées par l’utilisateur et elles
    prennent le dessus sur celles de defaults si elles sont renseignées.
-   **tasks**: Sans grande surprise, c’est ici que vous référencerez vos
    tâches.
-   **files**: Tous les fichiers étant destinés à être traités par le
    module copy seront placés ici.
-   **templates**: Idem que copy, mais cela concerne les fichiers du
    module template.
-   **meta**: Il y a ici plusieurs usages, notamment dans le cas de
    rôles publiés sur Galaxy. Dans notre cas, on référencera ici les
    dépendances à d’autres rôles.
-   **tests**: Prépare l'environnement de test (playbook de test et
    fichier d'inventaire) pour simuler le rôle.

Mise à part le dossier **tasks** qui est impératif, tous les autres ne
sont pas indispensables. Leur utilisation dépend des actions que l'on
souhaite réaliser.

#### Exemple de rôle

Pour cet exemple, on va utiliser notre rôle, webservices, précédemment
initialisé, pour monter un serveur web NGINX avec des modules php
installés.
```
    $ cat tasks/main.yml

    #************* Partie NGINX ***********************
    #Ajout du dépôt NGINX Stable
    - name: "Ajout du dépôt NGINX Stable"
      templates:
        src: nginx.repo.j2
        dest: /etc/yum.repos.d/nginx.repo
      when: ansible_distribution == "RedHat"
      tags: nginx

      # install nginx
    - name: "Install NGINX"
      package:
        name: nginx
        state: present
      notify: "Start NGINX"
      tags: nginx

    #************* Partie PHP ***********************
    # depot php remi
    - name: installation remi release
      yum:
        name: http://rpms.remirepo.net/enterprise/remi-release-7.rpm
        state: present
      tags: php

    - name: installation des paquets php
      yum:
        name: "{{ item }}"
        state: present
      with_items:
        - "{{ php_packages }}"
      tags: php
```
Dans le fichier `tasks/main.yml`, on installe le serveur web Nginx et
certains modules PHP comme suit :\
**Partie NGINX :** \
 Le dépôt Nginx est renseigné dans le fichier
**templates/nginx.repo.j2**. C'est un [template Jinja2](https://docs.ansible.com/ansible/latest/user_guide/playbooks_templating.html) dans lequel
certains paramètres seront obtenus dynamiquement.\
 A l'aide du module **templates**, on copie ce fichier sur la machine
cible si celle-ci est une distribution Redhat uniquement (**condition:
When**).\
 Ensuite, on installe le serveur NGINX. A la fin de l'installation, le
contenu du fichier **handlers/main.yml** (voir plus bas) est exécuté.

**Partie PHP :**
 On récupère les paquets PHP (php\_packages) directement à partir du
fichier **vars/main.yml** (voir plus bas). On fait une boucle
(l'instruction: With\_items) sur chacun des paquets (item) pour
l'installer.
```
    $ cat handlers/main.yml
    ---

    # Start NGINX
    - name: "tart NGINX"
      service:
        name: nginx
        state: started
        enabled: yes
      tags: nginx

    # Reload NGINX
    - name: "Reload NGINX"
      service:
        name: nginx
        state: reloaded
      tags: nginx

    $ cat templates/nginx.repo.j2

    [nginx]
    name=nginx repo
    description=nginx stable repo
    {% set os='rhel'}
    {% if ansible_distribution != 'RedHat' %}
     os='centos'
    {% endif %}
    baseurl=http://nginx.org/packages/{{ os }}/{{ ansible_distribution_major_version }}/$basearch/
    gpgcheck=0
    enabled=1

    $ cat vars/main.yml

    php_packages:
      - php
      - php-fpm
      - php-bz2
      - php-curl
      - php-dom
      - php-fileinfo
      - php-gd
      - php-iconv
      - php-json
      - php-mbstring
      - php-mysql
      - php-session
      - php-soap
      - php-sockets
      - php-sqlite3
```

#### Exécution du rôle

On ne pas exécuter directement un rôle comme on le fait avec les
playbooks. Pour exécuter un rôle, on crée un playbook, dans lequel, on
appelle le rôle.\
 Pour notre exemple, on va créer un playbook nommé ***webservices.yml***
dans le dossier playbooks (ou dans la racine du répertoire Ansible).\
 Le contenu de fichier ressemble à ceci:
```
    ---

    - hosts: webservers
      roles:
        - webservices
```
On peut exécuter le playbook par la suite :
```
    $ ansible-playbook webservices.yml [options]
```
Dans notre exemple, on suppose que le fichier d'inventaire est hosts, la
connexion aux machines se fait par clé SSH. Donc les options sont
facultatives.


Variables
---------

Nous avons rapidement pu voir les variables et les templates dans les
parties précédentes. Ils sont d'une grande utilité et facilite la vie de
l'administrateur pour l'automatisation des tâches.


### Types de variables

Ansible permet d'utiliser deux types de variables :
-   les variables sur l'environnement du serveur;
-   et les variables utilisateurs.

Les variables sur l'environnement du serveur (machine cible) sont
appelées des **facts**. Elles sont nombreuses et on peut les retrouver
sur la documentation du site ou à l'aide du module **setup** fourni par
Ansible.
```
    $ ansible localhost -m setup

    localhost | SUCCESS => {
        "ansible_facts": {
            "ansible_all_ipv4_addresses": [
                "192.168.220.17",
                "172.17.0.1"
            ],
            "ansible_all_ipv6_addresses": [],
            "ansible_apparmor": {
                "status": "disabled"
            },
            "ansible_architecture": "x86_64",
            "ansible_bios_date": "04/05/2016",
            "ansible_bios_version": "6.00",
            "ansible_cmdline": {
                "BOOT_IMAGE": "/vmlinuz-3.10.0-693.5.2.el7.x86_64",
                "LANG": "fr_FR.UTF-8",
                "crashkernel": "auto",
                "quiet": true,
                "rd.lvm.lv": "rhel/swap",
                "rhgb": true,
                "ro": true,
                "root": "/dev/mapper/rhel-root"
            },

         ...

         "ansible_distribution": "RedHat",
            "ansible_distribution_file_parsed": true,
            "ansible_distribution_file_path": "/etc/redhat-release",
            "ansible_distribution_file_variety": "RedHat",
            "ansible_distribution_major_version": "7",
            "ansible_distribution_release": "Maipo",
            "ansible_distribution_version": "7.4",

         ...
```
D'autres variables peuvent êtres définies par les utilisateurs; les
variables utilisateurs. Celles-ci peuvent être définies dans plusieurs
endroits :

-   Dans les playbooks, sous les sections **vars** (comme vu plus haut)
    ou **vars_prompt ou vars_files :**
```
    ---
    - hosts: webservers
      vars:
        http_port: 80
        max_clients: 200
      vars_files:
        - /vars/external_vars.yml
```
Ici, deux variables ont été définies: `http_port` et `max_clients`.\
 **Remarques:** Le nom des variables doit contenir uniquement des
lettres, des nombres et des underscores.\
 Les variables définies sous `vars_prompt` sont demandées à l'utilisateur
(au prompt).\
 Avec `vars_files`, on récupère des variables contenues dans un fichier.

-   Dans les roles, dans les dossiers **vars** ou **default:**
```
    $ cat vars/main.yml

    php_packages:
      - php
      - php-fpm
      - php-bz2
      - php-curl
      ...
```
Ici, on crée une variable `php_packages` contenant une liste d'items
(paquets).\
 **Remarque :** Les variables définies dans le dossier vars prévalent sur
celles définies dans default.

-   Dans les fichiers d'inventaire ou les fichiers contenus dans les
    dossiers `host_vars` ou `group_vars` :
```
    $ sudo cat /etc/ansible/hosts

    host1 http_port=80 maxRequestsPerChild=808
    host2 http_port=303 maxRequestsPerChild=909

    [webservers:vars]
    ntp=example.ntp.domaine.fr
    proxy=http://exampe.proxy.domaine.fr:3128
```

**Remarque :**
 - Pour les groupes de machines (ici webservers), on définit les
variables pour le groupe à l'aide du mot clé `vars`. Ainsi toutes les
variables situées sous `[webservers:vars]` sont uniquement pour ce groupe
de machines.
 - Si on ne souhaite pas trop surcharger les fichiers d'inventaire, pour
chaque groupe de machines, on peut créer, dans le dossier
`group_vars`, un fichier du même nom que le groupe de machine en
question et y ajouter les variables pour ce groupe. Ainsi pour le groupe
`webservers`, on va créer le fichier `group_vars/webservers` et y
ajouter ces lignes :
```
    ntp: example.ntp.domaine.fr
    proxy: "http://exampe.proxy.domaine.fr"
```

- Pour chaque machine particulière, pour laquelle on veut définir des
variables, on peut procéder de la même manière que pour les groupes de
machines. Cependant, on utilise le dossier `host_vars`. Par exemple :
```
    $ cat host_vars/host1

    http_port:80
    maxRequestsPerChild:808
```

### Utilisation des variables

Pour utiliser une variable, on met le nom de la variable entre accolades
{{ }}: Exemples :
```
    - pour les facts: {{ ansible_distribtion }}
    - pour les variables utilisateurs: {{ http_port }}
```
Les variables peuvent être utilisées dans plusieurs endroits :
-   dans un **playbook** (conditions when, boucle `with_items`, section `vars`,
    ...). Exemple :
```
    $ cat tasks/main.yml
    ...
    - name: "Ajout du dépôt NGINX Stable"
      templates:
        src: nginx.repo.j2
        dest: /etc/yum.repos.d/nginx.repo
      when: ansible_distribution == "RedHat"
    ...
    - name: installation des paquets php
      yum:
        name: "{{ item }}"
        state: present
      with_items:
        - "{{ php_packages }}"
    ...
```

  **Remarque:** Pour les conditions, on n'utilise pas les accolades `{{
}}`
-   dans les fichiers YAML des dossiers vars et default. Exemple :
```
    $ cat default/main.yml

    domain: exemple.domaine.fr

    http_proxy: "http://exemple.proxy.domaine.fr"

    host_name: "{{ inventory_hostname }}"
    proxy: "{{ http_proxy }}"
    host_ip: "{{ ansible_eth0.ipv4.address }}" #ou "{{ ansible_eth0["ipv4"]["address"] }}"
```
-   dans les templates (voir en bas)

Templates
---------

Les templates sont des fichiers qui peuvent être dynamisés, c'est à dire
qu'on peut certaines données peuvent être obtenues de manière
automatique.\
 Les templates Ansibles utilisent la sysntaxe [Jinja2](http://jinja.pocoo.org/).

### Quelques syntaxes jinja
```
    * {% ... %}: Pour les conditions et les boucles.<br>
    * {{ ... }}: Pour les expressions.<br>
    {% set ....}: Pour définir une variable
```
**Exemples :**
```
    #Conditions

    {% if ansible_distribution is Debian %}
       {{ ansible_eth0.ipv4.address }}
    {% endif %}

    #boucle for
    {% for host in groups['app_servers'] %}
       {{ hostvars[host]['ansible_eth0']['ipv4']['address'] }}
    {% endfor %}
```

### Exemples de templates

Nous avons vu un modèle de template précédemment :
```
    $ cat templates/nginx.repo.j2

    [nginx]
    name=nginx repo
    description=nginx stable repo
    {% set os='rhel'}
    {% if ansible_distribution != 'RedHat' %}
     os='centos'
    {% endif %}
    baseurl=http://nginx.org/packages/{{ os }}/{{ ansible_distribution_major_version }}/$basearch/
    gpgcheck=0
    enabled=1
```
Dans cet exemple, on crée le dépôt en fonction du système cible.
Certains paramètres de fichier sont obtenus dynamiquement.\
 En effet, la ligne ci-dessus varie selon l'OS et la version de la
distribution du système.
```
    baseurl=http://nginx.org/packages/{{ os }}/{{ ansible_distribution_major_version }}/$basearch/
```
Pour un autre exemple, on va créer un fichier de configuration apache
virtuel (`vhost`). Ce fichier s'adaptera en fonction des variables
définies dans `default/main.yml`.\
 Ainsi, on n'aura plus besoin de modifier le fichier vhost directement:
pour ce faire on modifier directement les paramètres (`http_port`,
`domain`,...) directement dans default/main.yml.
```
    $ cat templates/vhosts.conf.j2

    <VirtualHost *:{{ http_port }}>
        ServerAdmin webmaster@{{ domain }}
        ServerName {{ domain }}
        ServerAlias www.{{ domain }}
        DocumentRoot /var/www/{{ domain }}

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        <Directory /var/www/{{ domain }}/>
           Options -Indexes +FollowSymLinks
           AllowOverride All
        </Directory>
    </VirtualHost>
```
A l'exécution du playbook, les paramètres domain et `http_port` seront
remplacés par leurs valeurs.

### Utilisation des templates

Comme on l'a vu précedemment, on peut utiliser un template à l'aide du
module `templates` fourni par Ansible. C'est comme une opération de
copie, donc ça nécessite de préciser la source et la destination. A la différence du module copy, les variables seront remplacées automatiquement!
```
    cat tasks/main.yml
    ...
      templates:
        src: nginx.repo.j2
        dest: /etc/yum.repos.d/nginx.repo
```
