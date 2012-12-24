sasl2-bin 2.1.25.dfsg1-6 (utils)
================================

[saslauthd][acasă] —  un server de autentificare **SASL** ce implementează standardul **RFC 2222**.

[acasă]: http://www.cyrusimap.org/


Instalare pachete
-----------------

    HOSTu:~# apt install sasl2-bin libsasl2-modules
    The following extra packages will be installed:
      db-util db5.1-util

Notă: **apt** este un alias pentru `apt-get --purge` dar poți utiliza `aptitude` la fel de bine.


Configurare serviciu
--------------------

Editează fișierul de configurare `/etc/default/saslauthd` pentru a activa serviciul și modulele de autentificare utilizate:

    START=yes
    MECHANISMS="pam getpwent shadow"

Pentru utilizare cu `postfix` setează și locația prizei de legătură:

    OPTIONS="-c -m /var/spool/postfix/var/run/saslauthd"


Fapte cunoscute
---------------

### W: „cannot connect to saslauthd server: Permission denied”

Aceast mesaj poate apărea în jurnalul unui serviciu (ex. postfix).
Doar conturile din grupul `sasl` se pot conecta la priza de legătură cu serviciul **SASL**, locația fiind speficată prim parametrul `-m` (fără `/mux` la sfârșit).
