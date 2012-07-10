ntpdate :: 1:4.2.6.p2+dfsg-1+b1 (net)
=====================================

`ntpdate` este o unealtă pentru sincronizarea automată a timpului cu un server „NTP” public sau local.


Instalare pachete
-----------------

    HOSTu:~# apt remove chrony ntp ntpdate+

Notă: **apt** este un alias pentru `apt-get` dar poți utiliza `aptitude` la fel de bine.


Configurare serviciu
--------------------

#### Zona locală: UE sau US

    ZONE="de"
    ZONE="us"

#### Care sunt serverele „NTP” interogate pentru sincronizare?

    NTPSERVERS="0.${ZONE}.pool.ntp.org 1.${ZONE}.pool.ntp.org 2.${ZONE}.pool.ntp.org"

    TIME=$(getent hosts time | awk '{ print $NF }')
    if [ -n "$TIME" ]; then
      NTPSERVERS="$TIME $NTPSERVERS"
    fi

#### Generare număr aleator pentru minutul de sincronizare

    MM=60
    while [ $MM -ge 60 -o $MM -lt 20 ]; do
      MM=$(dd if=/dev/urandom count=1 2> /dev/null | cksum | cut -c"2,6")
    done

### Script „cron” pentru sincronizare automată a timpului

    cat > /etc/cron.d/ntpdate <<__EOF__
    MAILTO=root
    PATH=/sbin:/bin:/usr/sbin:/usr/bin

    #minute hour  dom mon dow  user      command
    $MM      */6     *   *   *  root      [ -x /usr/sbin/ntpdate ] && (pidof ntpd > /dev/null || /usr/sbin/ntpdate -s $NTPSERVERS) > /dev/null
    __EOF__
