# mirall-repositoris
Revisió actualitzada del projecte de l'alexm. Agraïments a l'alexm i al wagafo.

Servidor de miralls per a festes d'instal·lació
===============================================

Maquinari
---------

*   Un llapis USB o un CD amb l'instal·lador d'Ubuntu Server, segons si el portàtil té unitat de CD.
*   Un ordinador portàtil que pugui arrencar un disc dur del port USB i que tingui connexió WiFi.
*   Un disc dur extern amb connexió USB (com a mínim de 500 GB) i preferiblement de 2.5" perquè no calgui carregar la font d'alimentació.
*   Un cable USB per endollar el disc dur extern en un port USB del portàtil.
*   Un cable de xarxa creuat o un de directe, si el portàtil és capaç de fer crossover automàticament (la majoria d'ordinadors moderns ja tenen aquesta característica).

Programari
----------

*   Ubuntu Server
*   lvm2
*   dnsmasq
*   iptables
*   apt-mirror
*   apache2

Instal·lació bàsica del servidor
--------------------------------

*   Endolleu el disc dur extern en un port USB del portàtil amb el cable USB.
*   Arrenqueu el portàtil i feu que s'iniciï la instal·lació de l'Ubuntu Server des del llapis USB o del CD, segons s'escaigui.
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

La configuració de xarxa suposa que la interfície **eth2** és la de cable i la **eth3** la WiFi.

Definiu les variables d'entorn per les interfícies de xarxa:

    export LAN=eth2
    export WAN=eth3

Adapteu i copieu la configuració de les interfícies de xarxa:

    sudo cp -b interfaces /etc/network/interfaces

Instal·leu el paquet **dnsmasq**, adapteu i copieu la configuració a **/etc/dnsmasq.conf** (vegeu el paràmetre *no-dhcp-interface=eth2*):

    sudo cp -b dnsmasq.conf /etc/dnsmasq.conf

Instal·leu el paquet **apt-mirror** i executeu les ordres següents per preparar els miralls (utilitzarem el directori **/srv/mirror** que té muntat el volum pels miralls):

    mv ~apt-mirror/* /srv/mirror/
    rmdir ~apt-mirror
    ln -sf ../../srv/mirror ~apt-mirror
    cd ~apt-mirror/mirror
    sudo ln -s ftp.caliu.cat archive.ubuntu.com
    sudo ln -s archive.ubuntu.com es.archive.ubuntu.com
    sudo ln -s archive.ubuntu.com ad.archive.ubuntu.com
    sudo ln -s archive.ubuntu.com us.archive.ubuntu.com
    sudo ln -s archive.ubuntu.com rsync.archive.ubuntu.com

Instal·leu el paquet **apache2** i executeu les ordres següents per servir els mirralls:

    cd /var/www/html
    sudo ln -s ../../spool/apt-mirror/mirror/changelogs.ubuntu.com changelogs
    sudo ln -s ../../spool/apt-mirror/mirror/archive.ubuntu.com/ubuntu ubuntu

Copieu a */usr/local/bin* els guions que necessitareu més endavant:

    sudo cp mirror-list mirror-upgrader mirror-changelogs mirror-nat /usr/local/bin
    sudo chmod 0755 /usr/local/bin/mirror-*

Copieu a */usr/local/etc* la configuració de quines distribucions ha d'incloure el mirall:

    sudo cp mirrors.conf /usr/local/etc/mirrors.conf
    sudo chmod 0644 /usr/local/etc/mirrors.conf

Actualització dels miralls
--------------------------

Obtingueu una IP pel cable:

    sudo dhclient $LAN

Genereu la llista de fonts del mirall:

    /usr/local/bin/mirror-list | sudo tee /etc/apt/mirror.list

Actualitzeu el mirall:

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

Editeu l'ordre **mirror-list** i afegiu la nova distribució a la llista ***dists***.

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

    export LAN=eth2
    export WAN=eth3

Poseu en marxa la interfície LAN:

    sudo ip address add 10.0.0.10/8 dev $LAN

Encengueu el dnsmasq:

    sudo service dnsmasq restart

Encengueu l'apache:

    sudo service apache2 restart

Si voleu fer NAT amb la WiFi:

    sudo /usr/local/bin/mirror-nat

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
