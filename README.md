Servidor de miralls per a festes d'instal·lació
===============================================
Revisió actualitzada del projecte de l'@alexm. Agraïments a l'@alexm i al @wagafo.


Maquinari
---------

*   Un llapis USB o un CD amb l'instal·lador d'Ubuntu Server o de Debian, segons si el portàtil té unitat de CD.
*   Un ordinador portàtil que pugui arrencar un disc dur del port USB i que tingui connexió WiFi.
*   Un disc dur extern amb connexió USB (es recomana 1TB) i preferiblement de 2.5" perquè no calgui carregar la font d'alimentació.
*   Un cable USB per endollar el disc dur extern en un port USB del portàtil.
*   Un cable de xarxa creuat o un de directe, si el portàtil és capaç de fer crossover automàticament (la majoria d'ordinadors moderns ja tenen aquesta característica).


Programari
----------

*   Debian o Ubuntu Server
*   dnsmasq
*   iptables
*   apt-mirror
*   apache2
*   wireless-tools
*   wpasupplicant
*   pxelinux
*   tftpd-hpa
*   tftp-hpa
*   xinetd


Instal·lació bàsica del servidor
--------------------------------


### Ubuntu Server

*  Endolleu el disc dur extern en un port USB del portàtil amb el cable USB. 
*  Arrenqueu el portàtil i feu que s'iniciï la instal·lació de l'Ubuntu Server des del llapis USB o del CD, segons s'escaigui.
*  Escriviu el nom de màquina: **MirallUbuntaire** (s'utilitza al **dnsmasq.conf**).
*  Feu una instal·lació normal fins que arribeu a la selecció del disc.
*  La taula de particions ha de contenir 3 particions primàries:
   *  La primera de format ext2 per al /boot amb mida d’uns 500MB.
   *  La segona de format btrfs per al sistema.
   *  La terecera de tipus linux-swap per l’espai d’intercanvi amb mida d’uns 7GB.
*  Quan l'instal·lador us demani quins serveis voleu instal·lar, indiqueu-li només les utilitats bàsiques del sistema.
*  Instal·leu el GRUB, deixeu que acabi la instal·lació i reincieu per iniciar el sistema.

*  Munteu la partició principal:
   
   ```
   sudo mount /dev/sda2 /mnt
   cd /mnt
   ```

*  Per defecte Ubuntu crea dos subvolums: l’arrel i el home. Com que no interessa tenir el home separat feu (la nomenclatura      està pensada per x64):

   ```
   sudo mv @home/* @/home
   sudo btrfs sub delete @home
   sudo mv @ @64
   sudo btrfs sub create @miralls
   sudo mkdir snapshots
   ```

*  Elimineu la línia que muntava @home a **/etc/fstab**, modifiqueu la línia que munta l’arrel i afegiu la línia del subvolum dels    miralls:
   
   ```
   UUID=XXXXXX-YYYYYY / btrfs defaults,noatime,compress=lzo,subvol=@64 0 1
   /dev/sda2 /srv/mirror btrfs defaults,noatime,compress=lzo,subvol=@miralls 0 2
   ```

*  Actualitzeu el GRUB i reinicieu:

   ```
   sudo update-grub2
   sudo reboot
   ```


### Debian

*  Endolleu el disc dur extern en un port USB del portàtil amb el cable USB.
*  Inicieu el sistema amb un *live CD*. En aquest cas utilitzo Lubuntu.
*  Instal·leu les **btrfs-tools** i el **gparted**:

   ```
   sudo apt install btrfs-tools gparted
   ```
   
*  Des del Gparted creeu 3 particions primàries: la primera d’uns 500MB amb format **ext2**, la segona d’uns 7GB amb tipus            **linux-swap** i la tercera amb l’espai que sobra i amb format **btrfs**.
*  Creeu el directori /mnt/btrfs_partition i munteu-hi la partició btrfs:

   ```
   sudo mkdir /mnt/btrfs_partition
   sudo mount /dev/sda2 /mnt/btrfs_partition
   ```
	
*  Creeu un subvolum pel sistema i un pels miralls (la nomenclatura està pensada per x64).

   ```
   cd /mnt/btrfs_partition
   sudo btrfs subvolume create @64
   sudo btrfs subvolume create @miralls
   ```
	
*  Creeu un directori per imatges del disc:

   ```
   sudo mkdir snapshots
   ```
	
*  Per defecte el sistema arrel té l'ID=5, per tant si voleu veure els subvolums creats i els seus IDs feu:

   ```
   sudo btrfs subvolume list .
   ```

*  Com que voleu que l'instal·lador de Debian instal·li el sistema dins d'un subvolum feu:

   ```
   sudo btrfs subvolume set-default <ID> /mnt/btrfs_partition
   ```
	
*  Reinicieu el PC i inicieu el CD de Debian.
*  Escriviu el nom de màquina: **MirallUbuntaire** (s'utilitza al **dnsmasq.conf**).
*  Feu una instal·lació normal fins que arribeu a la selecció del disc. Allà escolliu **Manual**.
*  A la taula de particions han d’aparèixer les 3 particions creades anteriorment.
   *  Seleccioneu la d’ext2, indiqueu el punt de muntatge **/boot** i habiliteu-la per l’arrancada.
   *  Seleccioneu la btrfs, indiqueu que no es formati i escolliu el punt de muntatge **/**.
   *  Comproveu que la tercera partició està marcada com a intercanvi.
*  Quan l'instal·lador us demani quins serveis voleu instal·lar, indiqueu només les utilitats bàsiques.
*  Instal·leu el GRUB, deixeu que acabi la instal·lació i reincieu per iniciar el sistema.
*  Dins el fitxer **/etc/fstab** editeu la línia del / i afegiu la dels miralls (com a root):

   ```
   UUID=XXXXXX-YYYYYY / btrfs defaults,noatime,compress=lzo,subvol=@64 0 1
   /dev/sda2 /srv/mirror btrfs defaults,noatime,compress=lzo,subvol=@miralls 0 2
   ```
   
*  Restareu el volum per defecte (com a root):

   ```
   mkdir /mnt/btrfsroot
   mount -o subvolid=5 /dev/sda2 /mnt/btrfsroot
   btrfs subvolume set-default 5 /mnt/btrfsroot
   ```
	
*  Actualitzeu el GRUB i reinicieu (com a root):

   ```
   update-grub2
   reboot
   ```


Serveis del servidor
--------------------

*   dnsmasq
    *   interfaces
    *   dnsmasq.conf
*   iptables
    *   mirror-nat
*   apt-mirror
    *   mirrors.conf
    *   mirror-list
    *   mirror-upgrader
    *   mirror-changelogs
*   apache2
*   tftp
    *   tftp


Configuració dels miralls
-------------------------

Instal·leu el paquet **apt-mirror** i executeu les ordres següents per preparar els miralls (utilitzarem el directori **/srv/mirror**):
   
    sudo mkdir -p /srv/mirror
    sudo mv ~apt-mirror/* /srv/mirror/
    sudo rm -r ~apt-mirror
    sudo ln -s /srv/mirror ~apt-mirror
    cd ~apt-mirror/mirror
    sudo mkdir -p ftp.caliu.cat/ubuntu
    sudo mkdir -p changelogs.ubuntu.com
    sudo ln -s ftp.caliu.cat archive.ubuntu.com
    sudo ln -s archive.ubuntu.com es.archive.ubuntu.com
    sudo ln -s archive.ubuntu.com ad.archive.ubuntu.com
    sudo ln -s archive.ubuntu.com us.archive.ubuntu.com
    sudo ln -s archive.ubuntu.com rsync.archive.ubuntu.com

Instal·leu el paquet **apache2** i executeu les ordres següents per servir els mirralls:

    sudo apt-get install apache2
    cd /var/www/html
    sudo ln -s /var/spool/apt-mirror/mirror/changelogs.ubuntu.com changelogs
    sudo ln -s /var/spool/apt-mirror/mirror/archive.ubuntu.com/ubuntu ubuntu

Copieu a */usr/local/bin* els guions que necessitareu més endavant:

    sudo cp mirall-repositoris/mirror-* /usr/local/bin
    sudo chmod 755 /usr/local/bin/mirror-*

Copieu a */usr/local/etc* la configuració de quines distribucions ha d'incloure el mirall:

    sudo cp mirall-repositoris/mirrors.conf /usr/local/etc/mirrors.conf
    sudo chmod 644 /usr/local/etc/mirrors.conf


Actualització dels miralls
--------------------------

Genereu la llista de fonts del mirall:

    /usr/local/bin/mirror-list | sudo tee /etc/apt/mirror.list

Actualitzeu el mirall: (pendent de revisar, salta un error, almenys amb Debian).

    sudo chown -R apt-mirror:apt-mirror /srv/mirror
    sudo chown -R apt-mirror:apt-mirror /var/spool/apt-mirror
    sudo su - apt-mirror -c apt-mirror
    sudo su - apt-mirror -c mirror-upgrader
    sudo su - apt-mirror -c mirror-changelogs

Si avorteu la sincronització del mirall i us cal esborrar el bloqueig, feu:

    sudo rm ~apt-mirror/var/apt-mirror.lock


Neteja dels miralls obsolets
----------------------------

Després d'una actualització o purga d'un mirall us pot interessar fer net:

    sudo su - apt-mirror -c ~apt-mirror/var/clean.sh


Introducció de nous miralls
---------------------------

Editeu el fitxer **mirrors.conf** i afegiu una nova línia amb la distribució. Atenció! La primera distribució de la llista també es baixarà com a NetBoot!


Configuració de la xarxa
---------------------------

La configuració de xarxa suposa que la interfície **eth0** és la de cable i la **wlan0** la WiFi.

Assegureu-vos de tenir instal·lat tant el **wireless-tools** com el **wpasupplicant**:

    sudo apt-get install wireless-tools
    sudo apt-get install wpasupplicant

Adapteu i copieu la configuració de les interfícies de xarxa i reïniciu-les:

    sudo cp -b mirall-repositoris/interfaces /etc/network/interfaces
    sudo /etc/init.d/networking restart

Instal·leu el paquet **dnsmasq**, adapteu i copieu la configuració a **/etc/dnsmasq.conf**:

    sudo apt-get install dnsmasq
    sudo cp -b mirall-repositoris/dnsmasq.conf /etc/dnsmasq.conf


Arrancar per xarxa
---------------------------

*Pendent servidor NTP.

Instal·leu el **tftp**, el **xinetd** i el **pxelinux**:

    sudo apt-get install pxelinux tftpd-hpa tftp-hpa xinetd

Creeu el directori **/var/lib/tftpboot** i baixeu els NetBoots:

    sudo mkdir /var/lib/tftpboot
    chmod +x mirall-repositoris/netboot-download
    sudo mirall-repositoris/netboot-download
    sudo cp -b mirall-repositoris/default /var/lib/tftpboot/pxelinux.cfg/default
    
Habiliteu el servidor:

    sudo cp -b mirall-repositoris/tftp /etc/xinetd.d/tftp
    sudo service xinetd restart
    sudo service tftpd-hpa stop
    sudo service xinetd restart
    sudo update-rc.d -f tftpd-hpa remove


Preparació de la festa
======================

Maquinari
---------

*   Un ordinador portàtil que pugui arrencar un disc dur del port USB i que pugui tenir connexió WiFi.
*   El disc extern dels miralls.
*   Un cable USB per endollar el disc dur extern en un port USB del portàtil.
*   Un cable de xarxa creuat o un de directe, si el portàtil és capaç de fer crossover automàticament (la majoria d'ordinadors moderns ja tenen aquesta característica).
*   Un commutador de xarxa amb ports suficients pel servidor i la resta d'ordinadors de la festa.
*   Un cable directe per cada ordinador que faci de client a la festa.


Servidor
--------

Exporteu les variables:
    
    export LAN=eth0
    export WAN=wlan0

Engegueu el dnsmasq:

    sudo service dnsmasq restart

Engegueu l'apache:

    sudo service apache2 restart

Engegueu el tftp:

    sudo service xinetd restart

Si voleu fer NAT amb la WiFi:

    sudo /usr/local/bin/mirror-nat
    

A vegades si connectes el disc dur extern a un altre ordinador no et detecta correctament les targetes de xarxa. Per arreglar-ho feu:

    sudo udevadm trigger
    sudo udevadm trigger --action=add
    
Si això no funciona elimineu el fitxer **/etc/udev/rules.d/70-persistent-net.rules**, reinicieu i després feu:

    sudo udevadm trigger --action=change


Clients
-------

*   Desactiveu els repositoris de fonts (sources).
*   Desactiveu els repositoris de tercers (canonical, medibuntu, etc.)
*   Desactiveu els repositoris de proposed.
*   Activeu algun respositori de la llista següent:
    *   http://archive.ubuntu.com/ubuntu
    *   http://ad.archive.ubuntu.com/ubuntu
    *   http://es.archive.ubuntu.com/ubuntu
    *   http://us.archive.ubuntu.com/ubuntu
    *   http://ftp.caliu.cat/ubuntu
