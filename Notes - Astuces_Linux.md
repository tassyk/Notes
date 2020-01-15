
### Liste de tous les fichiers setuid
      find / -perm /6000 -type f -exec ls -ld {} \; > setuid.txt

###  Enlever les droits setuid sur qq fichiers
    chmod a-x /usr/bin{cmd1, cmd2}

###  session multi-fenetres
    screen

###  Le nombre d'occurrence d'une chaine dans un fichier
    for x in `grep "File does not exist" error_log | awk '{print $13}' | sort | uniq`; do \
    echo -n "$x : ", grep $x error_log | wc -l; done

###  commande d'impression
    lp

###  Alias SUDO
voir: User_Alias, Runas_Alice, Host_Aliace, Cmd_Aliace,...


###  Automatisation des tâches d'admin avec Makefile
Les Makefiles rendent toutes les opérations (as slmt gcc) plus rapides et plus faciles.

###  Trouver un nouveau nom de domaine (se terminant par "st") par force brute
    cat /usr/share/dict/words | grep 'st$' | sed 's/st$/.st/' | \
    while read i; do \
    (whois $i | grep -q '^No entries found') && echo $i; sleep 60; \
    done | tee liste_domaines_st.txt

###  Déterminer une utilisation intensive de votre système de fichier à l'aide d'un alias
- A placer dans .profile, puis a exécuter partout
      alias ducks='du -cks * | sort -rn | head -11'

###  Manipuler /proc
Afficher directement la table des processus et les variables système du noyau
- version du noyau
      $ cat verion
- Taille de la RAM utilisée
    $ ls -l kcore
- Partitions
      $ cat partitions
- Afficher la structure d'un processus (1)
    `$ cd 1
    $ ls -l`
- Afficher l'environnement du processus
      `$ cat self/environ |tr '\0' '\n'`
-  Comparer la sortie de ps avec la table des processus
      `$ ls -d /proc/* | grep [0-9] | wc -l; ps ax | wc -l`

###  Manipuler symboliquement des processus via procps
- Bloquer l'utilisateur sur le terminal pts/2
        `$ skill -STOP pts/2`
- Pour les débloquer
      `  $ skill -CONT pts/2`
- Modifier les priorités des processus de 'utilisateur '(à 5)
      `$ snice +5 utilisateur`
- Tuer un processus d'un user
      `$ skill -KILL robert  bash ou $ pkill -KILL -u robert bash`
  NB: Même effet, mais pkill permet plus de précision

- Afficher les PID uniquement
      ` $ pgrep httpd3211 `
NB: pgrep fonctionne comme pkill mais n'affiche que les PID

- Analyser la mémoire vive et afficher les stats
      $ vmstat

###  Gérer les ressources systèmes par process
- Limiter le nombre max de fichiers à utliser par un processus
      $ ulimite -f 100

###  Fermer l'accès complètement au départ d'un utilisateur
- 1ère étape judicieuse: blocage du compte par un mot de passe
      $ passwd -l *utilisateur*
- Autre étape bien connue: changer le shell de connexion de l'utilisateur en un autre qui se termine immédiatement:
      $ chsh -s /bin/true *utilisateur*
NB: En cas d'une auth par clé, l'utilisateur peut effectuer par contre une redirection ssh pour se connecter. Par exeple:
      $ ssh -f -N -L8000:priv.intranet.server.com:80 ancien.server.com
- Supprimer alors les fichiers ci-dessous s'ils existent et empêcher le d'utiliser sa clé ssh.
       ~utilisateur/.ssh/authorized_keys*
       ~utilisateur/.shosts
       ~utilisateur/.rhosts
- Vérifier si l'utilisateur avait un compte sudo
      $ visudo
-  Vérifier si l'utilisateur avait des tâches cron ou at ou était en cours d'exécuter un processus
      $ crontab -u utilisateur -e
      $ atq
      $ ps aux | grep -i ^utilisateur
      $ skill -KILL utilisateur
- Pourrait-il exécuter des programmes cgi? Quid de PHP ou autres langages?
      $ find ~utilisateur/public_html/ -perm +111
      $ find ~utilisateur/public_html/ -name '*.php*'
- Avait il configuré la redirection de mail?
      $ less ~utilisateur/.forward
      $ grep -C utilisateur /etc/mail/aliases
- Est-il propriétaire de fichiers dans d'autres emplacements?
      $ find -user utilisateur > ~root/utilisateur-files.report
- Une manière sûre et rapide de s'assurer que tous les fichiers de l'utilisateur sont invalidés est de réaliser un mv
        $ mv /home/utilisateur /home/utilisateur.supprime

###  Utiliser un système de fichier *tmpfs* pour le répertoire */tmp*
- Le système de fichiers tmpfs permet de monter des répertoires en mémoire virtuelle, cela est particulièrement intéressant pour le répertoire /tmp car accessible par tous.

###  Allouer des quotas
- Laissez les utilisateurs auto-gérer les quotas

###  Augmenter la taille de swap sans utiliser de partition dédiée
- Augmenter la taille de swap sur son système de fichiers de façon à pouvoir lancer plus d'applications, que prévu, ou des applications avec des jeux de données beaucoup plus importants

###  Mettre en place un environnement "chrooté"
- Déplacez une arborescence de fichiers sur une autre partition avec *chroot*

###  Dissimuler des fichiers
- Dissimuler des fichiers pour améliorer la sécurité du système


# Sauvegarde
- Sauvgarder avec tar sur SSH
      $ tar zcvf - /home | ssh pinky "cat > inky-home.tgz"
- Utiliser rsync sur SSH
      ~# rsync -ave ssh greendome:/home/ftp/pub/ /home/ftp/pub/
- Archiver avec Pax
  PAX peut créer une archive tar comme une archive cpio

# Firewall Iptables
#### Masquerade IP simple
- masquerade (NAT): 2 lignes suffisent
      # echo "1" > /proc/sys/net/ipv4/ip_forward
      # iptables -t nat -A POSTROUTING -o $EXT_IFACE -j MASQUERADE

#### Contre deni-de-service TCP
- Détection DDoS
      iptables -t nat -N syn-flood
- Limite de 12 connexions par seconde
      iptables -t nat -A syn-flood -m limit --limit 12/s --limit-burst 24  -j RETURN
      iptables -t nat -A syn-flood -j DROP
- Contrôle d'attaque par déni de service
      iptables -t nat -A PREROUTING -i $EXT_IFACE -d $DEST_IP -p tcp --syn -j syn-flood
##### Redirection vers un proxy Squid
- Redirection flux tcp à destination du port 80 vers le proxy
      `iptables -t nat -A PREROUTING -i $INT_IFACE -p tcp --dport 80 -j REDIRECT --to-port 3128`
# Création de tunnels
#### Encapsualtion IPIP
- Créer des tunnels avec le module ipip (# modprobe ipip)
- Ou créer des tunnels avec le module ip_gre (# modprobe ip_gre)

#### Utiliser vtun sur SSH pour contourner NAT
- Connecter deux réseaux ensemble à l'aide e vtun et d'une seule connexion SSH (# modprobe tun)

# Surveiller des tâches avec watch
- Utiliser watch pour exécuter répétitivement n'importe quelle commande et afficher les résultats
- Exple:
      `  # watch 'ps ax | grep tar'`

# Contrôler les fichiers et les sockets ouverts avec lsof
- Visualiser aisément quels fichiers, répertoires et sockets sont maintenus ouverts par vos processus en cours.

# Surveiller le réseau avec ngrep
- Observer qui fait quoi, avec un grep pour votre interface réseau
- Exemple:
      `# ngrep -q GET`

# Obtenir des stats en temps réel
#### Ntop pour les stats réseaux
- Grâce à ntop, sachez en permanence qui fait quoi sur votre réseau

#### Surveiller le trafic web avec httptop
- Sachez dans la seconde qui se connecte le plus fréquemment à votre serveur avec httptop
- Exemple:
      `$ httptop -f vhost /usr/local/apache/logs/combined-log `
#### Tracer des programmes avec strace et ltrace
- Utilisez les outils strace et ltrace pour vous sortir de situations inextricables
- Exemple (prog marchepas):
      `$ strace -o strace-marchepas.log marchepas `

# Utilisation SSH
#### Ouvrir une session en mode turbo
- Ouvrir des sessions encore plus vite depuis la ligne de commande
- 1/  Créer un lien symbolique vers un serveur et exécuter ssh dessus rien qu'en appelant le nom du serveur
        `$ cat ssh-vers`
            #!/bin/sh
            ssh `basename $0` $*
        # ajouter ssh-vers au PATH
        `$ cd bin`
        `$ ln -s ssh-vers serveur1`
        # Exemple d'appel
      `  $ serveur1 uptime`
- 2/ Autre méthode: créer des alias mais fastidieux à maintenir
        `$ alias alice="ssh alice"`

#### Transfert e ports sur SSH
- Sécuriser le trafic de réseau vers des ports arbitraires avec le transfert de port SSH (tunnel SSH)
- Exemple:
        $ ssh -f -N -L110:serveur-message:110 -l utilisateur serveur-message

        # Autoriser d'autres clients à se connecter au port transféré
        $ ssh -f -g -N -L8000:localhost:80 10.42.4.6

# Télécharger et convertir des images
- Récupérer des images dépuis Internet et suvegarder-ler sous d'autres formats
- Exemples:
        $ curl http://lien/images.jpg | convert -img.png
        $ wget -O - http://lien/images.jpg | convert -img.png
        # Afficher l'image
        $ display img.png
- NB: CURL et WGET le proxy également
        `$ curl -x nom_ou_address_proxy_http[:port] [-U identifiant[:mot_de_passe]] ...`

        `$ wget -e http_proxy=[protocole://]nom_ou_adresse_proxy[:port] [--proxy-user=identifiant] [--proxy-passwd=mot_de_pase]`

# Créer des livrets
- Transformer et imprimer des fichiers PostScript sous la forme de livrets
- see: http://linuxprinting.org
#### Quelques commandes PostScript
- See: commande **psbook**
        `$ psbook source.ps`
        # 2 pages par feuille
        `$ psbook source.ps | psnup -2`
        # 2 pages par feuille et recto-verso
        `$ psbook source.ps | psnup -2 | psset -t`
        # Evoyer à l'imprimante
        `$ psbook source.ps | psnup -2 | psset -t | lpr`
        # Que les pages paires (-e) ou impaires (-o)
        `$ psbook source.ps | psnup -2 | psset -t | psselect -o | lpr`
#### Convertir vos PostScript
- Créer un cachier à partir d'un doc qui n'est pas au format PostScript -> convertir
      # sysntaxe :
      $ commande_de_conversion | commande_impression

- NB commande_de_conversion:
      # pdf to PostScript
      `$ pdftops source.pdf - `
      # pdf to PostScript à partir d'Adobe Acrobate
      `$ cat source.pdf | acroread -toPostScript`
      # txt to PostScript
      `$ enscript -M A4 --word-rap -o - source.txt`

# Traitements par lots sur un annuaire
- Chainer les instructions ldapsearch, perl (sed ou awk) et ldapmodify pour effectuer des traitements par lot sur un annuaire LDAP
- syntaxe:
        `$ ldapsearch ....|perl -pe '...' |ldapmodify`
- NB: ldapmodify prend en entrée un flux de données au format slapd.


# Faire fonctionner BIND dans une prison chroot
- L'idée est d'isoler le processus named du reste du système par l'utilisation judicieuse de chroot
- Ainsi en cas de compromission du service, l'attaquant aura encore du travail.

# Surveiller la santé de MySQL avec mtop
- Afficher les processus de MySQL en temps réel dans une format similaire à celui de tp

# Servir plusieurs sites avec la même variable DocumentRoot
- A travers une utiisation ingénieuse de **mod_rewrite**, plusieurs sites peuvent partager une variable DocumenRoot et apparaître comme totalement indépendants les uns des autres.
-
