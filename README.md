# PXE-SRV-SETUP
- OS : Debian
- Services: TFTP, DHCP, HTTP,NFS
- IPXE : Fichier BOOT

# Installation des Services

```python
# Dnsmasq : (contient les services TFTP,DHCP,DNS)
# nfs-kernel-server (Service NFS)
# nginx (Service HTTP)
apt install -y dnsmasq  nfs-kernel-server nginx 
```

# Création du Fichier BOOT iPXE

```python
# Installation de Git pour cloner le repository du projet iPXE

apt install -y git

# Téléchargment du repository de iPXE depuis GitHub
cd /opt/

# Téléchargment du repository de iPXE depuis GitHub
git clone https://github.com/ipxe/ipxe.git

```

## Création du fichier "chain.ipxe"
```python

# Pour aller dans le répertoire Src
 cd /opt/ipxe/src

# Créer le fichier "chain.ixpe" en ajoutant le contenu en index 1
 nano chain.ipxe
```

> index 1
```python

#!ipxe

dhcp

# charge le menu depuis le Service HTTP
chain http://{Adresse du Serveur}/menu.ipxe
```


## Création du fichier "menu.ipxe"
```python
# le fichier menu.ipxe est le Menu Personnalisable du iPXE

# Créer le fichier "menu.ixpe" en ajoutant le contenu en index 2
 nano  /var/www/html/menu.ipxe
```

>index 2
```cpp
#!ipxe

set menu-timeout 9000000
set submenu-timeout ${menu-timeout}
isset ${menu-default} || set menu-default item3

# change le chemin de logo.png par votre image a vous
console --picture logo.png


menu
item --gap --           -------------SÉPARATION 1----------------
item item1            	Item 1
item --gap --           -------------SÉPARATION 2----------------
item item2      	Item 2
item item3		Item 3
item shell              Shell iPXE
item exit              	Exit

choose --timeout ${menu-timeout} --default ${menu-default} target && goto ${target}

:item1
#Paramètres de démarrage pour item 1

:item2
#Paramètres de démarrage pour item 2

:item3
#Paramètres de démarrage pour item 3

:shell
shell

:exit
exit

```

## Optionnel - activation de la Personnaltion Menu
```python
# Suivre le Tuto
# https://github.com/ipxe/ipxe/discussions/293
```

## Compilation des fichiers de Boot

```python
cd /opt/ipxe/src 

apt-get install -y make build-essential zlib1g-dev binutils-dev

# Compilation fichier de Boot  64Bits
make bin-x86_64-efi/ipxe.efi EMBED=chain.ipxe

# Compilation fichier de Boot  32Bits
NO_WERROR=1 make bin-i386-pcbios/undionly.kpxe EMBED=chain.ipxe

# Copie des fichiers de  Boot 32 et 64 bits vers /var/lib/tftpboot/
cp  bin-i386-pcbios/undionly.kpxe bin-x86_64-efi/ipxe.efi  /var/lib/tftpboot/ -vv 

```

## Configuration de Dnsmasq

```python
# Faire un backup du fichier dnsmasq.conf
mv /etc/dnsmasq.conf /etc/dnsmasq.conf.bak

# Editer le fichier "dnsmasq.conf" en ajoutant le contenu en index 3
 nano  /etc/dnsmasq.conf
```


>index 3
```python
# Doesn't work  as a DNS server:
port=0

# Logs lots of extra information about DHCP transactions.
log-dhcp

# Sets the root directory for files available via TFTP.
enable-tftp
tftp-root=/var/lib/tftpboot/

# The boot filename, Server name, Server Ip Address
dhcp-boot=undionly.kpxe,,192.168.100.253

# Disables re-use of the DHCP servername and filename fields as extra
# option space. That's to avoid confusing some old or broken DHCP clients.
dhcp-no-override

# inspect the vendor class string and match the text to set the tag
dhcp-vendorclass=BIOS,PXEClient:Arch:00000
dhcp-vendorclass=UEFI32,PXEClient:Arch:00006
dhcp-vendorclass=UEFI,PXEClient:Arch:00007
dhcp-vendorclass=UEFI64,PXEClient:Arch:00009


# PXE menu.  The first part is the text displayed to the user.  The second is the timeout, in seconds.
pxe-prompt="Booting PXE SERVER", 1

# The known types are x86PC, PC98, IA64_EFI, Alpha, Arc_x86,
# Intel_Lean_Client, IA32_EFI, BC_EFI, Xscale_EFI and X86-64_EFI
# This option is first and will be the default if there is no input from the user.
pxe-service=X86PC, "Boot to PXE SERVER (BIOS)",undionly.kpxe
pxe-service=X86-64_EFI, "Boot to PXE SERVER (UEFI)",ipxe64.efi

# DHCP PROXY dans le cas ou vous avez deja un serveur DHCP sur votre Réseau
dhcp-range=192.168.100.1,proxy
```

## Configuration du Service NFS

```python 
# ajouter l'emplacement a partager via NFS
nano /etc/exports

# La ligne de l'emplacement a ajouter
 /var/www/html/   nfs-client-ip(rw,sync,no_subtree_check)

```
## Démarrage des Services 

```python
# Démarrage du service Dnsmasq
systemctl enalbe dnsmasq
systemctl start dnsmasq 

# Démarrage du service HTTP
systemctl enalbe ngnix
systemctl start ngnix

# Démarrage du service NFS
systemctl restart nfs-server
systemctl status nfs-server

```
## Example de Menu Configuration d'un Système Linux

```python

#!ipxe
set server 192.168.100.253
set menu-timeout 9000000
set submenu-timeout ${menu-timeout}
isset ${menu-default} || set menu-default item3

menu
item --gap --           -------------Operating Systems----------------
item item1                           Live-Raizo
item shell              Shell iPXE
item exit               Exit

choose --timeout ${menu-timeout} --default ${menu-default} target && goto ${target}

:item1
#Paramètres de démarrage pour item 1
kernel http://${server}/Live-Raizo/live/vmlinuz
initrd http://${server}/Live-Raizo/live/initrd.img
imgargs vmlinuz initrd=initrd.img root=/dev/nfs boot=live netboot=nfs nfsroot=${server}:/var/www/html/Live-Raizo locales=fr_FR.UTF-8 keyboard-layouts=fr hostname=raizo username=user user-fullname=Live-Raizo-User ip=frommedia ethdevice-timeout=60 
boot
```

## Source 

```python


# Guide de configuration Ipxe (Methodologie) et autres...
https://doc.ubuntu-fr.org/ipxe

# Configuration Serveur NFS 
https://www.atlantic.net/dedicated-server-hosting/how-to-install-and-configure-nfs-server-on-debian-11/


```
