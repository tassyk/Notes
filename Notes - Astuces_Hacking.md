# Création d'un labo virtuel
- Livre: **Hacking**
Un labo virtuel pour auditer et mettre en place des contre-mesures

## Administrations de Proxmox en CLI
#### Les commandes utiles : vzctl et qm
- Il existe 2 cmd principales:
  - **vzctl** pour contrôler les machines d'OpenVZ
  - **qm** pour les machines KVM
- Sauvegarde / Restauration: **vzdump** et **vzrestore**
- Exemples:
  - Lister les machines
        `$ vzlist -a
        $ qm list`
  - Démarrer/stoper une machine
       `$ vzctl start/stop *VMID*
        $ qm start/stop *VMID*`
  - Entrer dans une VM
        `$ vzctl enter *VMID*`
  - Créer une VM
        `$ vzctl create *VMID* --ostemplate *vm_template_name*\
       --hostname *host_name* --name *vm_name*`
- see: vzctl/qm --help

## Serveur DHCP
- Installation
      $ aptitude install isc-dhcp-server
- Fichier de conf:
  - /etc/dhcp/dhcp.conf
  - La conf type pour chaque VM ayant une IP fixe est:
          `host nom_machine {
            hardware ethernet adresse_MAC;
            fixed-adress adresse_IP;
          }`
  - La conf type pour les VM ayant une IP dynamique:
          `Subnet adresse_reseau netmask masque_reseau {
                range ip_première_machine ip_derniere_mchine;
                option routers ip_passerelle;
          }`
- Start / Stop
      $ service isc-dhcp-server start|stop|restart

## Installation et configuration OpenVPN
##### Sur le serveur
- Installer les paquets
      `$ aptitude install openvpn`
- Copier les fichiers de config
      `$ sudo mkdir /etc/openvpn/easy-rsa/
      $ sudo cp -r /usr/share/doc/openvpn/examples/easy-rsa/2.0/* /etc/openvpn/easy-rsa `
      `$ sudo chown -R $USER /etc/openvpn/easy-rsa`
- Editer le fichier vars pour adapter les variables
- Génerer les certificats (.crt) et les clés (.key)
      `$ cd /etc/openvpn/easy-rsa/
       $ source vars
       $ ./clean-all
       $ ./build-dh
       $ ./pkitool -initca
       $ ./pkitool -server server
       $ sudo openvpn -genkey -secret keys/ta.key`
- Déplacer les certificats et les clés
      `$ sudo cp -r keys/* /etc/openvpn/`
- (Facultatif) Chrooter le processus openvpn (avec la commande chroot)
      $ sudo mkdir /etc/openvpn/jail
      $ sudo mkdir /etc/openvpn/clientconf
- Editer le fichier de conf: server.conf
      ...
      # Reseau
      server 10.8.0.0 255.255.255.0
      ...
      # Securite
      chroot /etc/openvpn/jail
      ...
- Lancer le serveur
      `$ /etc/init.d/openvpn start `
- Configurer le serveur pour qu'il joue le rôle de routeur entre l'interface VPN (tun0) et l'interface physique eth0 et "natter" les adresses en 10.8.0.X
  - Autoriser le forwarding: ajouter la ligne ci-dessous dans /etc/sysctl.con
      net.ipv4.ip_forward=1
  - Réaliser la translation NAT (iptables)
      `$ iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE`
  - Fixer la règle même après le redémarrage
      `$ iptables-save > /etc/iptables.rules`

- Générer les clés et certificat des clients
      `$ cd /etc/openvpn/easy-rsa
      $ source vars
      $ ./build-key myclient`
- Créer un répertoire pour le client et y copier les fichiers
      $ mkdir /etc/openvpn/clientconf/myclient
      $ cp /etc/openvpn/ca.crt /etc/openvpn/ta.key keys/myclient.crt keys/myclient.key /etc/openvpn/clientconf/myclient
- Config le fichier de conf du client
      $ cd /etc/openvpn/clientconf/myclient
      $ nano client.conf
        # Client
        client
        dev tun
        ...
        # Adresse IP du serveur vpn
        remote 192.168.1.2 443
        ...
        # Cles
        ca ca.crt
        cert myclient.crt
        key myclient.key
        tls-auth ta.key 1
        ...
- Compresser ces fichiers et l'envoyer au client
      `$ cd /etc/openvpn/clientconf/myclient
      $ zip myclient.zip *.*`

#### Sur le client
- Décompresser l'archive et se placer sur le répertoire
      `$ unzip myclient.zip`
- Se connecter au serveur
      `$ openvpn client.conf`
NB:
- S'il y a un proxy, ajouter la ligne ci-dessous dans client.conf
      http_proxy adresse_ip_proxy port_proxy passwd.txt basic
- passwd.txt comportera en première ligne le login et en duxième le password.

- Sous windows, renommer client.conf en client.ovpn avant de lancer la connexion

## Installation serveur FTP (VsFTP)
- Installation
      `$ aptitude install vsftpd`
- Configuration: /etc/vsftpd.conf
      ...
      # Interdire les actions anonymes
      anonymous_enable=NO
      anon_upload_enable=NO
      anon_mkdir_write=NO
      ...
      # Autres sécu  
      async_abor_enable=NO
      ascii_upload_enable=NO
      ascii_download_enable=NO
      chroot_local_user=YES
      chroot_list_enable=NO
      ...
- Start
      `$ /etc/init.d/vsftpd start`
- NB:
  - Les utilisateurs ont accès à leur répertoire personnel(mais il faut qu'ils aient été créés sur le serveur)
  - Les actions sont enregistrées dans /var/log/vsftpd.log
- Créer un compte pour tous (rep_common)
    $ adduser rep_common
    $ chmod -R 755 /home/rep_common
    $ chown rep_common:rep_common -R /home/rep_common
- Créer pour chaque utilisateur un répertoire rep_common
    `$ mkdir /home/user/rep_common
    $ chown user:user /home/user/rep_common
    chmod 755 /home/user/rep_common`
- Créer un lien entre le rep_common de chaque user avec le répertoire common
    `$ nano /etc/fstab
        /home/rep_common /home/user/rep_common auto bind,defaults 0 0`
- Monter ce répertoire
      `$ mount  /home/user/rep_common`
- Nous pouvons maintenant se connecter au serveur via un client FTP (FileZilla)

## Serveur Asterix (VoIP)
- Config: sip.conf & extensions.conf
##### sip.conf
- Tous les comptes utilisateurs sont créés dans ce fichier
      `[user1]
      type=friend
      host=dynamic
      user=user1
      secret=password
      context=default`

- **type=friend** permet d'appeler et d'être appelé
- **host=dynamic** l'adresse IP du client est définie par DHCP
- **secret** le password est en clair. Il faut utiliser plutôt **md5secret**
  - Pour md5secret, la structure c'est <user>:<realm>:<secret>
  - Le réalm est **asterisk** par défaut
  - Pour génerer l'empreinte md5 de l'utilisateur *user*:
      `$ echo -n "user:asterisk:password" | md5sum`

##### Extensions.conf
- Ce fichier permet de définir le plan de numérotation (dialplan)
- C'est de la forme
      `[default]
      exten => 555,1,Dial (SPIP/user1)
      exten => 555,1,Dial (SPIP/user2)`
  - 555: est le num d'appel
  - 1: est le num de séquence

##### Prise en compte des modifications
      `asterisk -rvvvvdddd
       reload
      dialplan reload`

#### Ajout de fonctions
###### Transfert d'appel
- Il faut activer ls tonalités DTMF (Dual-Tone Muti-Frequency) dans sip.conf:
      `dtmfmode = rfc2833`
- Pour activer le trafsert, ajouter les options "tT" dans une exten (Extensions.conf)
    exten => 555,2,Dial(SIP/user1, ,tT)
- Pour effectuer le transfert, il faut appuyer sur le signe **#** durant une conversation

##### Mise en attente
- Elle permet de mettre en pause une communication
- Il faut activer, là aussi, les tonalités DTMF
- Pour activer la mise en attente, il suffit d'ajouter la ligne suivante dans le context [dfault] du fichier extensions.conf:
      `include=>parkedcalls`
- Pour mettre son interlocuteur en attente, composer **#700**

###### Messagerie vocale
- Pour mettre la voix en Français, il faut le spécifier lors de la compilation d'Asterisk
      `$ make menuselect`
- Il faut ensuite, ajouter dans le contexte [general] de sip.conf:
      `language=fr`
- Nous pouvons alors créer une boite vocale dans /etc/asterisk/voicemail.conf, dans [default]:
      `100 => 1010, user1, user1@ascissi.net`
NB:
  - 100 est le numéro de téléphone auquel on associe a boite vocale
  - 1010 est le mot de passe de la messagerie
  - user1@ascissi.net est l'adresse mail de l'utilisateur
- Activer la boite vocale dans sip.conf
      `exten => 100,1,Dial(SIP/user1, 10)
      exten => 100,2, Voicemail(b101)
      exten => 100,3,Hangup `
###### Configuration du MTA pour la messagerie vocale
- Exemple MTA: Exim4 (MTA par défaut de Debian)
- Configuration
      `dpkg-reconfigure exim4-config`

## Configuration d'un client SIP (Ekiga)
- Ekiga est un logiciel de téléphonie SIP et de visioconférence
- Installer Ekiga sur le client
      `$ aptitude install ekiga`
- Suivre les instructions
- Une fois la config terminée, il ne reste plus qu'à se onnecter sur le serveur via son **IP**
- Pour ajouter un compte manuel, aller dans **Editions - Comptes** puis **Comptes - Ajouter un compte SIP**
NB: Le registraire sera l'IP du serveur SIP
