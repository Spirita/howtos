# Noyau Linux

Auteur : Cyrille Toulet, <cyrille.toulet@gmail.com>

Version : 14.10.17.1

## Introduction

On veut compiler un nouveau noyau !

On commence par se renseigner sur la version du noyau actuellement utilisée 
sur notre machine.

```sh
uname -v
```

Chez moi c'est la version 3.16 ;~)

**NB :** Toutes les manipulations qui suivent supposent qu'on est root.

## Compilation d'un noyau linux

On installe quelques paquets :

 - **build-essential :** pour make et gcc
 - **libncurses-dev :** parce qu'on en a besoin (mouhouhouhahaha)
 - **linux-source :** qui sont les fichiers sources du kernel

``` sh
aptitude install build-essential libncurses-dev linux-source
```

On se place dans le répertoire des sources et on décompresse les sources 
(à adapter en fonction du noyau) :
```sh
cd /usr/src
tar -xvf linux-source-3.16.tar.xz
cd linux-source-3.16
```

Puis on lance la configuration :

``` sh
make defconfig
make menuconfig
```

**NB 1 :** On peut ajouter un suffixe au nom du noyau pour le différencier au 
boot :
```
 General setup  --->
  (-nom-de-ma-version-custom-trop-cool) Local version - append to kernel release
```

**NB 2 :** La configuration par défaut créée par make defconfig est très légère 
(voir trop), ne comptons pas booter dessus ;-) 

Pour un premier test, on peut copier la configuration actuelle :
```sh
cp /boot/config-3.16-2-amd64 /usr/src/linux-source-3.16/.config
```

Il reste à compiler et installer le noyau :

``` sh
make -j 4
make -j 4 modules_install
make install
```

Une fois le make install effectué, on peut rebooter et choisir notre nouveau 
noyau, si ça ne tourne pas rond, il suffit de rebooter sur l'ancien !

## Suppression d'un noyau

Pour supprimer un noyau, il faut enlever ses fichiers de */boot*. 
Par exemple :
```sh
mv /boot/*-nom-de-ma-version-custom-trop-cool /tmp/
```

Puis comme un grand, on regenere les fichiers de conf de grub :
```sh
update-grub
```

## Exemple de configuration du noyau

Voici un exemple des options modifiées par rapport à la configuration par 
défaut pour construire un noyau linux serveur et pouvant être utilisé avec 
xen :
```
General setup  --->
  (-tiir) Local version - append to kernel release
  [*] Automatically append version information to the version string

Processor type and features  --->
  [*] Paravirtualized guest support  --->
      [*]   Xen guest support

  Preemption Model (No Forced Preemption (Server))  --->
      (X) No Forced Preemption (Server)

  [*] Machine Check / overheating reporting
  [*]   Intel MCE features 
  [ ]   AMD MCE features  

  <M> /dev/cpu/microcode - microcode support               
  [*]   Intel microcode patch loading support
  [ ]   AMD microcode patch loading support

  [ ] Old style AMD Opteron NUMA detection

Bus options (PCI etc.)  --->
  < > PCCard (PCMCIA/CardBus) support  --->
  < > Support for PCI Hotplug  --->

[*] Networking support  --->
    Networking options  ---> 
      <M>   IP: tunneling
      <M> 802.1d Ethernet Bridging
      [ ] QoS and/or fair queueing  --->

  [ ]   Amateur Radio support  --->

  [ ]   Wireless  --->

  < >   RF switch subsystem support  --->

Device Drivers  --->
  [*] Block devices  --->
      <M>   Xen block-device backend driver

  <*> Serial ATA and Parallel ATA drivers  --->
      [ ]   ATA SFF support

  [*] Multiple devices driver support (RAID and LVM)  --->
      < >   RAID support

  [ ] Macintosh device drivers  --->

  [*] Network device support  --->
      <M>     Universal TUN/TAP device driver support

      [*]   Ethernet driver support  --->
         Décochez tout et ne conservez que
         [*]   Intel devices
            <*>     Intel(R) PRO/1000 PCI-Express Gigabit Ethernet support

      < >   FDDI driver support 

      [ ]   Token Ring driver support  --->

      [ ]   Wireless LAN  --->

      <M>   Xen backend network device 

  Graphics support  --->
      < > /dev/agpgart (AGP Support)  --->

      [ ] Backlight & LCD device support  --->

  <*> Sound card support  ---> 
      <*>   Advanced Linux Sound Architecture  --->
         [*]   PCI sound devices  --->
             <*>   Intel HD Audio  --->
                Utilisez lsmod pour voir les codecs utilisés
                [*]   Build Realtek HD-audio codec support
                [*]   Build HDMI/DisplayPort HD-audio codec support
                [*]   Enable generic HD-audio codec parser

         [ ]   USB sound devices  --->

  [*] HID Devices  --->
          Special HID drivers  --->
                Tout désactiver

  [*] USB support  --->
      [ ]     USB device filesystem (DEPRECATED)

      < >   USB Monitor

      < >   USB Printer support

  [ ] LED Support  --->

  [*] IOMMU Hardware Support  --->
      [ ]   AMD IOMMU support

File systems  --->
  Activez bien celui utilisé lors du formatage à l'installation 
  < > Ext3 journalling file system support
  <*> The Extended 4 (ext4) filesystem 

  Même sans lecteur CD, peut être utile pour monter des images iso
  CD-ROM/DVD Filesystems  --->
     <M> UDF file system support

  DOS/FAT/NT Filesystems  ---> 
     <M> NTFS file system support

  [*] Network File Systems  --->
     <M>   NFS server support

  Partition Types  --->
     Supprimez tout sauf
     [*]   PC BIOS (MSDOS partition tables) support
     [*]   EFI GUID Partition support

  -*- Native language support  --->
     <*>   NLS ISO 8859-15 (Latin 9; Western European Languages with Euro)

Security options  --->
     [ ] NSA SELinux Support
```
