snmpd :: 5.4.3~dfsg-2 (net)
===========================

[snmpd][home] este un server ce răspunde la cereri «SNMP».

[home]: http://net-snmp.sourceforge.net


Instalare pachete
-----------------

    HOSTu:~# apt install snmp snmpd  snmp-mibs-downloader

Ultimul pachet nu este necesar în majoritatea cazurilor.

Notă: **apt** este un alias pentru `apt-get --purge` dar poți utiliza `aptitude` la fel de bine.


Configurare serviciu
--------------------

#### Folosește «syslog», închide «stdin/out/err»

sed -i "s%-Lsd%-LS5d%" /etc/default/snmpd

Am ales nivelul 5 pentru a evita „info” și „debug”.

#### Configurație minimă

    DOMAIN="nume-domeniu-extern.TLD"
    LOCATION="Craiova, RO"
    cat > /etc/snmp/snmpd.conf <<__EOF__
    rocommunity  public
    sysName      $(hostname -f)
    sysLocation  $LOCATION
    sysContact   root@${DOMAIN:-$(dnsdomainname)}
    disk         /   20%
    includeAllDisks  6%
    __EOF__

Opțional, se poate restricționa accesul astfel:

    #rocommunity public  localhost
    #rocommunity public  10.10.10.0/24

#### Start serviciu

    HOSTu:~# service snmpd restart
