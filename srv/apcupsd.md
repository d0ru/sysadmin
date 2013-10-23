apcupsd 3.14.10-2 (admin)
=========================

[Apcupsd][acasă] — un serviciu pentru monitorizarea unui „UPS” marca [APC][producător].

[acasă]: http://www.apcupsd.com/
[producător]: http://www.apc.com/

Scurt glosar:

* **UPS** — „Uninterruptible power supply”


Instalare pachete
-----------------

Cerință pre-instalare: `apache2`

    HOSTu:~# apt install apcupsd apcupsd-doc-  apcupsd-cgi
    The following extra packages will be installed:
      libgd2-noxpm libjpeg8 libpng12-0

Notă: **apt** este un alias pentru `apt-get --purge` dar poți utiliza `aptitude` la fel de bine.


Configurare serviciu „MASTER”
-----------------------------

Editează fișierul `/etc/apcupsd/apcupsd.conf` să conțină:

    UPSCABLE smart
    UPSTYPE usb
    #DEVICE /dev/ttyS0
    LOCKFILE /var/lock
    SCRIPTDIR /etc/apcupsd
    PWRFAILDIR /etc/apcupsd
    NOLOGINDIR /etc
    ONBATTERYDELAY 6
    BATTERYLEVEL 20
    MINUTES 2
    TIMEOUT 0
    ANNOY 300
    ANNOYDELAY 6
    NOLOGON disable
    KILLDELAY 0
    NETSERVER on
    NISIP 0.0.0.0
    NISPORT 3551
    EVENTSFILE /var/log/apcupsd.events
    EVENTSFILEMAX 4096
    UPSCLASS sharemaster
    UPSMODE share
    STATTIME 86400
    STATFILE /var/log/apcupsd.status
    LOGSTATS off
    DATATIME 0
    UPSNAME APC_no1
    #SELFTEST 336

Activează serviciul:

    sed "s|^#*ISCONFIGURED=.*$|ISCONFIGURED=yes|" -i /etc/default/apcupsd
    service apcupsd start


Configurare serviciu „SLAVE”
----------------------------

Editează fișierul `/etc/apcupsd/apcupsd.conf` să conțină:

    UPSCABLE smart
    UPSTYPE net
    DEVICE upsmaster.DOMAIN.tld:3551
    LOCKFILE /var/lock
    SCRIPTDIR /etc/apcupsd
    PWRFAILDIR /etc/apcupsd
    NOLOGINDIR /etc
    ONBATTERYDELAY 6
    BATTERYLEVEL 20
    MINUTES 3
    TIMEOUT 0
    ANNOY 300
    ANNOYDELAY 6
    NOLOGON disable
    KILLDELAY 0
    NETSERVER on
    NISIP 127.0.0.1
    NISPORT 3551
    EVENTSFILE /var/log/apcupsd.events
    EVENTSFILEMAX 4096
    UPSCLASS shareslave
    UPSMODE share
    STATTIME 86400
    STATFILE /var/log/apcupsd.status
    LOGSTATS off
    DATATIME 0
    #UPSNAME UPS_IDEN
    #SELFTEST 336

Activează serviciul:

    sed "s|^#*ISCONFIGURED=.*$|ISCONFIGURED=yes|" -i /etc/default/apcupsd
    service apcupsd start
