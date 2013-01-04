clamav 0.97.6+dfsg-1 (utils)
============================

[ClamAV][acasă] — un anti-virus pentru Linux/UNIX.

[acasă]: http://www.clamav.net/


Instalare pachete
-----------------

    HOSTu:~# apt install clamav clamav-daemon libclamunrar6
    The following extra packages will be installed:
      clamav-base clamav-freshclam libclamav6 libltdl7 libtommath0

    HOSTu:~# apt install arc arj bzip2 cabextract lzma lzop nomarch \
      pax p7zip ripole sharutils tnef unzip zip zoo

Notă: **apt** este un alias pentru `apt-get --purge` dar poți utiliza `aptitude` la fel de bine.

#### Forțează prima actualizare a bazei de semnături ClamAV

    HOSTu:~# freshclam

Motorul **ClamAV** poate fi monitorizat în timp real astfel:

    HOSTu:~# clamdtop


Configurare serviciu
--------------------

### Setează opțiuni prin reconfigurare pachet

    HOSTu:~# dpkg-reconfigure clamav-base

    Maximum stream length (unit Mb) allowed:       0
    Maximum directory depth that will be allowed:  20

Am setat parametrul `StreamMaxLength` la 0 pentru a scana toate fișierele indiferent de dimensiune.
Limitarea dimensiunii ar trebui să se facă în serviciul din față (ex. postfix).

Pentru setări suplimentare editează `/etc/clamav/clamd.conf` să conțină:

    LogFacility LOG_MAIL
    DetectBrokenExecutables Yes

### Setează actualizarea automată a bazei de semnături ClamAV

    HOSTu:~# dpkg-reconfigure clamav-freshclam

    Number of freshclam updates per day:     60
    Should clamd be notified after updates?	 <Yes>

Notă: modulul [*Google Safe Browsing*][safebrowsing] necesită actualizarea bazei de date la un interval de 30 minute.

[safebrowsing]: http://www.clamav.net/lang/en/faq/faq-safebrowsing/

Pentru setări suplimentare editează `/etc/clamav/freshclam.conf` să conțină:

    LogFacility LOG_MAIL
    SafeBrowsing Yes

Configurația curentă poate fi verificată astfel:

    HOSTu:~# clamconf -n
