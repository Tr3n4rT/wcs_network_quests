# INSTALLATION D'UN SERVEUR DHCP avec Linux

## Prérequis

Pour cette procédure nous allons utiliser un poste Linux Debian 12 qui nous servira de serveur DHCP et un poste Linux Ubuntu 24 qui nous servira de client.

Tapper la commande `ip a` nous permet d'afficher les interfaces de notre poste serveur. Relevez le nom de l'interface avec laquelle vous allez accueillir les requêtes DHCP. 
Nous allons relever l'interface __enp0s3__ pour la suite de l'installation.

![configuration de départ](https://github.com/Tr3n4rT/wcs_network_quests/blob/main/INSTALLATION-DHCP-LINUX/images/serveru-dhcp-config-(interface).png)


## Installation

### Installation du package DHCP

Installez le paquet __isc-dhcp-server__ avec la commande :\
`sudo apt install isc-dhcp-server`

## Configuration du DHCP


1. Attribuez une adresse ip statique à votre poste serveur DHCP en éditant le fichier __interfaces__ :\
`sudo vi /etc/network/interfaces`

Ajoutez les lignes si dessous en attribuant à votre serveur DHCP les adresses IP, mask de sous réseaux, et passerelle correspondante à votre configuration réseau :

```bash
auto enp0s3
iface enp0s3 inet static
adress 172.20.02
netmask 255.255.255.0
gateway 172.20.0.1
```

2. Editez le fichier __isc-dhcp-server__ pour définir les interfaces avec lesquelle le serveur DHCP disctibura les requêtes DHCP :\
`sudo vi /etc/default/isc-dhcp-serveur`

Dans ce fichier nous rentrons le mot clé __INTERFACES__ suivi du nom de notre interface __enp0s3__ relevé précédemment : 
`INTERFACES="enp0s3"`

![configuration interface](https://github.com/Tr3n4rT/wcs_network_quests/blob/main/INSTALLATION-DHCP-LINUX/images/interface-configuration.png)

3. Indiquez la configuration distribuée (plages, adresse de réseaux, passerelle...) par le serveur DHCP en éditant le fichier __dhcpd.conf__, avec les paramêtres correspondant à votre configuration :
`sudo vi /etc/dhcp/dhcpd.conf`

  - Si votre serveur DHCP est le seul serveur DHCP de votre réseau, décommentez la ligne "_authoritative_" en supprimant le symbole __#__ juste devant.
  - Ajustez la durée du bail en éditant les lignes suivantes :

```bash
default-lease-time 600;
max-lease-time 7200;
```

  - Ajoutez les lignes suivante en adaptant les adresses réseaux à votre configuration :
```bash
subnet 172.20.0.0 netmask 255.255.255.0 {
        option routers  172.20.0.1;
        option subnet-mask  255.255.255.0;
        option domain-searsh    "example.lan";
        option domain-name-servers  172.20.0.1;
        range   172.20.0.100    172.20.0.200;
}
```

## Configuration de l'interface réseau Ubuntu

1. Relevez le nom de votre interface réseau après avoir entrée la commande `ip a` dans un terminal.

2. Enfin de connecter notre nouvel machine virtuel Ubuntu à un réseau, il faut indiquer à l'outil de gestion du réseau __netplan__ l'interface réseau que nous allons utiliser à chaque démarrage pour intérroger le serveur DHCP.\
Entrez la commande : `sudo vim /etc/netplan/*.yaml` afin de configurer le fichier .yaml.\
Si aucune interface n'est configuré pour le dhcp rentrez les ligne suivantes dans le fichier, en prenant soin de rentrer le nom de l'interface et de son adresse MAC correspondant à votre configuration.
```bash
# Let NetworkManager manage all devices on this system
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    enp0s3:
      dhcp4: true
      match:
        macaddress: 08:00:27:56:bc:b8
      mtu: 1450
      set-name: ens8
```
appliquez la configuration en entrant la commande : `sudo netplan apply`

Si votre serveur DHCP sur votre machine Debian est activé sur le réseau, vous pouvez constater grâce à la commande `ip a` qu'une adresse ip correspondante au réseau configuré précédemment à bien été attribuer à votre ordianateur Ubuntu

![ip-dynamic-ubuntu](https://github.com/Tr3n4rT/wcs_network_quests/blob/main/INSTALLATION-DHCP-LINUX/images/attrib-dynamic-ubuntu.png)

## Configurer le serveur DHCP pour attribuer une adress ip statique
 
1. Avec l'adresse mac de l'interface réseaux de notre poste Ubuntu configuré si dessus, nous allons, depuis notre serveur DHCP, attribuer une adresse ip statique à cette interface en éditant le fichier __dhcpd.conf__.\
Entrez la commande sudo vi `/etc/dhcp/dhcpd.conf` sur votre serveur dhcp.\
Au dessous des lignes contenant la configuration réseau à distribuer, configuré précédemment, ajoutez les lignes suivantes correspondant à votre configuration réseau et à l'interface MAC du poste client auquel vous voulez attribuer une adresse ip fixe.
```bash
host ubuntu-24 {
    hardware ethernet 08:00:27:56:bc:b8
    fixed-adress 172.20.0.10
}
```
Sauvegarder votre fichier et quittez.\
Redémarrer le service dhcp de votre serveur avec la commande `systemctl restart isc-dhcp-server.service`

2. Sur votre poste client Ubuntu, redémarez le processus réseau afin d'appliquer la nouvelle configuration grâce à la commande `sudo netplan apply`\
On peu vérifier que l'adresse désormais attribuer à notre machine Ubuntu est bien celle que nous avons configuré.

![ip-statique-ubuntu](https://github.com/Tr3n4rT/wcs_network_quests/blob/main/INSTALLATION-DHCP-LINUX/images/ip-static-ubuntu.png)



