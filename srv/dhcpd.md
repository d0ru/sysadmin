isc-dhcp-server :: 4.1.1-P1-15+squeeze3 (net)
=============================================

Serviciul **DHCP** este utilizat pentru configurarea automată și dinamică a rețelei pe sistemele de calcul.


Configurare server DHCP
-----------------------

### Parametrii generali

    HOSTu:~# sed -i "s%^\(option domain-name\) .*$%\1 \"$(dnsdomainname)\";%" /etc/dhcp/dhcpd.conf

    HOSTu:~# NS="10.100.0.1, 10.100.0.2"
    sed -i "s%^\(option domain-name-servers\) .*$%\1 $NS;%" /etc/dhcp/dhcpd.conf

    HOSTu:~# sed -i "s%^\(default-lease-time\) .*$%\1 7200;%" /etc/dhcp/dhcpd.conf
    sed -i "s%^\(max-lease-time\) .*$%\1 86400;%" /etc/dhcp/dhcpd.conf

    sed -i "s%^#*\(authoritative;\)$%\1%" /etc/dhcp/dhcpd.conf
    sed -i "s%^#*\(log-facility local7;\)$%#\1%" /etc/dhcp/dhcpd.conf

    HOSTu:~# sed -i "s%^#*authoritative;$%&\nalways-broadcast on;%" /etc/dhcp/dhcpd.conf

### Adăugare subnet local

    HOSTu:~# cat >> /etc/dhcp/dhcpd.conf <<__EOF__

    subnet 10.100.0.0 netmask 255.252.0.0{
      interface eth0;
      range 10.100.200.1 10.100.200.254;
      option routers 10.100.0.1;
      option subnet-mask 255.252.0.0;
      option broadcast-address 10.103.255.255;
    }
    __EOF__

### Adăugare înregistrări statice

    HOSTu:~# cat >> /etc/dhcp/dhcpd.conf <<__EOF__

    # SERV group
    include "/var/cache/dhcp/SERV.conf";

    # PROF group
    include "/var/cache/dhcp/PROF.conf";
    __EOF__

Înregistrările statice sunt actualizate automat pe baza fișierului «hosts.list».

    HOSTu:~# cat > /etc/dhcp/hosts.list <<__EOF__
    #Group | Host | MAC address | Employee | Location | Description

    # SERV group
    # network subnet: nnn.nnn.nnn.nnn/mm
    SERV|ahile|14:fe:b5:ca:17:c9|root|229|Debian Linux: file server

    # PROF group
    # network subnet: 10.100+X.YZ.0/14 (XYZ = room)

    # Dynamic allocated throught DHCP for guests
    # network subnet: 10.100.200.0/24, 10.200.200.0/24

    # Reserved for VPN connections
    # network subnet: 10.100.100.0/24
    __EOF__

### Activare serviciu

    HOSTu:~# sed -i "s%^\(INTERFACES=\).*$%INTERFACES=\"eth0\"%" /etc/default/isc-dhcp-server

    HOSTu:~# update-dhcp-groups
    HOSTu:~# service isc-dhcp-server restart
