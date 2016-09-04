Servidor de miralls per a festes d'instal·lació
===============================================
Revisió actualitzada del projecte de l'@alexm. Agraïments a l'@alexm i al @wagafo.
*Pendent de confirmar tot el relatiu a les particions (LVM, particions separades, etc.). Mirar també amb btrfs i mirar si el servidor Debian no falla amb els repositoris de l'Ubuntu.

Maquinari
---------

*   Un llapis USB o un CD amb l'instal·lador d'Ubuntu Server o de Debian, segons si el portàtil té unitat de CD.
*   Un ordinador portàtil que pugui arrencar un disc dur del port USB i que tingui connexió WiFi.
*   Un disc dur extern amb connexió USB (es recomana 1TB) i preferiblement de 2.5" perquè no calgui carregar la font d'alimentació.
*   Un cable USB per endollar el disc dur extern en un port USB del portàtil.
*   Un cable de xarxa creuat o un de directe, si el portàtil és capaç de fer crossover automàticament (la majoria d'ordinadors moderns ja tenen aquesta característica).

Programari
----------

*   Ubuntu Server o Debian
*   lvm2
*   dnsmasq
*   iptables
*   apt-mirror
*   apache2
*   wireless-tools
*   wpasupplicant

Instal·lació bàsica del servidor
--------------------------------

*   Endolleu el disc dur extern en un port USB del portàtil amb el cable USB.
*   Arrenqueu el portàtil i feu que s'iniciï la instal·lació de l'Ubuntu Server o del Debian des del llapis USB o del CD, segons s'escaigui.
*   Feu una instal·lació normal fins que arribeu a la selecció del disc.
*   Trieu un particionat amb LVM sobre el disc dur extern que teniu endollat al port USB.
*   La taula de particions del disc ha de contenir 2 particions primàries:
    *   La primera de tipus **Linux** (83) per al **/boot** amb mida entre 250 MB.
    *   La segona de tipus **Linux LVM** (8e) per al sistema amb la resta del disc.
    *   La segona partició serà el volum físic d'un grup de volums anomenat **ubuntaires**, que conté aquests volums lògics:
        *   **root** per al sistema, amb 8 GB.
        *   **swap** per al fitxer d'intercanvi, amb 2 GB.
        *   **mirror** per als miralls, amb la mida que calculeu que us cal (uns quants centenars de GB). Per exemple, podeu muntar-lo al directori **/srv/mirror** o qualsevol altre que us agradi més.
    *   No assigneu tot l'espai disponible als miralls per si us cal ampliar algun dels altres volums lògics en algun moment.
    *   Formateu la partició de **/boot** i els volums lògics amb el sistema de fitxers ext3 o ext4.
*   Quan l'instal·lador us demani quin usuari voleu crear, indiqueu-li que es diu **ubuntaires** i la contrasenya **ubuntu.cat** (si us interessa que algú pugui connectar remotament a aquest ordinador, trieu una contrasenya més robusta).
*   Quan l'instal·lador us demani quins serveis voleu instal·lar, indiqueu-li que cap (en concret, desactiveu el servei SSH si voleu que ningú pugui connectar remotament).

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

Configuració dels miralls
-------------------------

També es dóna per suposat que no hi ha instal·lat el NetworkManager o el Wicd (les instal·lacions sense entorn gràfic no els instal·len).



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

    sudo cp mirror-* /usr/local/bin
    sudo chmod 755 /usr/local/bin/mirror-*

Copieu a */usr/local/etc* la configuració de quines distribucions ha d'incloure el mirall:

    sudo cp mirrors.conf /usr/local/etc/mirrors.conf
    sudo chmod 644 /usr/local/etc/mirrors.conf

Actualització dels miralls
--------------------------

Genereu la llista de fonts del mirall:

    /usr/local/bin/mirror-list | sudo tee /etc/apt/mirror.list

Actualitzeu el mirall: (pendent de revisar, salta un error, almenys amb Debian).

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

Editeu el fitxer **mirrors.conf** i afegiu una nova línia amb la distribució.

Configuració de la xarxa
---------------------------

La configuració de xarxa suposa que la interfície **eth0** és la de cable i la **wlan0** la WiFi.

Definiu les variables d'entorn per les interfícies de xarxa:

    export LAN=eth0
    export WAN=wlan0

Assegureu-vos de tenir instal·lat tant el **wireless-tools** com el **wpasupplicant**:

    sudo apt-get install wireless-tools
    sudo apt-get install wpasupplicant

Adapteu i copieu la configuració de les interfícies de xarxa i reïniciu-les:

    sudo cp -b interfaces /etc/network/interfaces
    sudo /etc/init.d/networking restart

Instal·leu el paquet **dnsmasq**, adapteu i copieu la configuració a **/etc/dnsmasq.conf**:

    sudo apt-get install dnsmasq
    sudo cp -b dnsmasq.conf /etc/dnsmasq.conf


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

Definiu les interfícies de xarxa:

    export LAN=eth0
    export WAN=wlan0

Engegueu el dnsmasq:

    sudo service dnsmasq restart

Engegueu l'apache:

    sudo service apache2 restart

Si voleu fer NAT amb la WiFi:

    sudo /usr/local/bin/mirror-nat
    

Pendent afegir el tema de les **/etc/udev/rules.d/70-persistent-net.rules** per compatibilitat amb diferents PCs.

    http://muzso.hu/2012/10/29/how-to-regenerate-the-etc-udev-rules.d-70-persistent-net.rules-file-on-debian-ubuntu
    http://unix.stackexchange.com/questions/255715/how-to-regenerate-70-persistent-net-rules-without-reboot
   

Clients
-------

*   Desactiveu els repositoris de fonts (sources).
*   Desactiveu els repositoris de tercers (canonical, medibuntu, etc.)
*   Desactiveu els repositoris de backports i proposed.
*   Activeu algun respositori de la llista següent:
    *   http://archive.ubuntu.com/ubuntu
    *   http://ad.archive.ubuntu.com/ubuntu
    *   http://es.archive.ubuntu.com/ubuntu
    *   http://us.archive.ubuntu.com/ubuntu
    *   http://ftp.caliu.cat/ubuntu
