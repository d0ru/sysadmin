squid3 3.1.20-2 (web)
=====================

[Squid][acasă] — un server proxy pentru web, cu suport pentru **FTP** și **HTTP**.

[acasă]: http://www.squid-cache.org/


Configurare serviciu
--------------------

### Dezactivare semnături „proxy”

    CONFIG="/etc/squid3/squid.conf"
    sed 's|^#* *\(forwarded_for\) .*$|\1 transparent|' -i $CONFIG
    sed 's|^#* *via o.*$|via off|' -i $CONFIG

Aceste semnături anunță prezența unui serviciu „proxy”, lucru nedorit în anumite cazuri.

### Permite accesul din rețeaua locală

    CONFIG="/etc/squid3/squid.conf"

    INTRANET="10.100.0.0/14 10.200.0.0/14"
    grep "^acl INTRANET" $CONFIG || \
      sed "s|^acl to_localhost dst .*$|&\nacl INTRANET src $INTRANET|" -i $CONFIG

    sed "s|^http_access allow localhost$|&\nhttp_access allow INTRANET|" -i $CONFIG

### Configurare autentificare «DIGEST»

Editează fișierul de configurarea să conțină parametrii pentru autentificare:

    auth_param digest program /usr/lib/squid3/digest_pw_auth /etc/squid3/pw_digest
    auth_param digest children 5
    auth_param digest realm Squid proxy-caching web server (HOST.DOMAIN.tld)
    auth_param digest nonce_garbage_interval 5 minutes
    auth_param digest nonce_max_duration 30 minutes
    auth_param digest nonce_max_count 50

Notă: porturile marcate `transparent`, `intercept` sau `tproxy` au autentificarea dezactivată.

Crează fișierul de configurare pentru autentificare:

    user=utilizator
    pass=parola0000
    echo "$user:$pass" >> /etc/squid3/pw_digest

Adaugă o listă de control pentru autentificarea „digest”:

    CONFIG="/etc/squid3/squid.conf"
    grep "^acl auth_digest" $CONFIG || \
      sed "s|^acl to_localhost dst .*$|&\nacl auth_digest proxy_auth REQUIRED|" -i $CONFIG

Permite accesul prin autentificare *digest*:

    grep "^http_access allow auth_digest" $CONFIG || \
      sed "s|^http_access allow localhost$|&\nhttp_access allow auth_digest|" -i $CONFIG
