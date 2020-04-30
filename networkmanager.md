---
Title: NetWorkManager
Type: Doc
Nature: Notes
Création: 30/04/2020
---

# Gestion des interfaces réseau via NetWorkManager
## Installation de NetworkManager
```
# installer le paquet
sudo yum install NetworkManager
# activer le service
systemctl start NetworkManager
systemctl enable NetworkManager
# Voir le status du service
systemctl status NetworkManager
```
## Interactions avec le NetworkManager
Via deux outils :
- `nmtui`: en mode graphique
- `nmcli`: en mode ligne de commande

### Gestion en mode graphique - NMTUI
- Installer le paquet tui : ` yum install NetworkManager-tui`
- `nmtui`: pour accéder à l'interface

### Gestion en ligne de commande - NMCLI
- sysntaxe : `nmcli OPTIONS OBJECT { COMMAND | help }`

**Quelques usages**:

- Afficher toutes les connexions : `nmcli connection show`
> Note : `--active` pour les connexions activées

- Voir le status des interfaces : `nmcli device status`
- Activer une interface (enp0s3): `nmcli con up id enp0s3`
- Désactiver une interface (enp0s3): `nmcli dev disconnect enp0s3`
- Editeur nmcli graphique : `nmcli con edit`
> Syntaxe: `nmcli con edit [id | uuid | path] ID`
> Exemple: pour éditer l'interface enp0s3, `nmcli con edit type ethernet con-name enp0s3`


## Sources
- [Redhat: NetworkManager, nmtui](https://access.redhat.com/documentation/fr-fr/red_hat_enterprise_linux/7/html/networking_guide/sec-networking_config_using_nmtui)
- [Redhat: NetworkManager, nmcli](https://access.redhat.com/documentation/fr-fr/red_hat_enterprise_linux/7/html/networking_guide/sec-using_the_networkmanager_command_line_tool_nmcli)
