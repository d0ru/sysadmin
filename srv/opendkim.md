opendkim 2.6.8-3 (mail)
=======================

[OpenDKIM][acasă] — autentificare **DKIM** în serviciul **MTA** (ex. postfix).

[acasă]: http://www.opendkim.org/

Scurt glosar:

* **DKIM** — „DomainKeys Identified Mail”, *RFC 5585*
* **ADSP** — „DKIM Author Domain Signing Practices”, *RFC 5617*
* **ATSP** — „DKIM Authorized Third-Party Signatures”, *RFC 6541*


Instalare pachete
-----------------

    HOSTu:~# apt install opendkim
    The following extra packages will be installed:
      libldns1 liblua5.1-0 libopendkim7 libunbound2 libvbr2

Notă: **apt** este un alias pentru `apt-get --purge` dar poți utiliza `aptitude` la fel de bine.


Generează cheia RSA pentru DKIM
-------------------------------

Perechea de chei **RSA** este asociată unui domeniu email și unui selector.

Pentru simplitate voi utiliza două variabile de mediu:

    DOMAIN=$(domainname)
    SELECTOR=$(hostname -s)

Generează o cheie **RSA** de 2048 biți:

    opendkim-genkey -b 2048 -d $DOMAIN -s $SELECTOR
    mv -v ${SELECTOR}.private dkim_${SELECTOR}.key

Alternativ, poți genera o cheie **RSA** de 2048 biți astfel:

    openssl genrsa -rand /dev/urandom -out dkim_$SELECTOR.key 2048

Extrage cheia publică **RSA** din cheia privată:

    openssl rsa -in dkim_$SELECTOR.key -pubout -outform PEM -out dkim_$SELECTOR.pem

Setează permisiunile fișierelor:

    chmod 0400 dkim_$SELECTOR.key
    chown opendkim:opendkim dkim_$SELECTOR.*
    mv -v dkim_$SELECTOR.* /etc/mail/


Actualizare record DNS pentru DKIM
----------------------------------

Extrage cheia publică **RSA** pe o singură linie:

    HOSTu:~# grep -v "^-" /etc/mail/dkim_${SELECTOR}.pem | tr -d '\n'; echo

Adaugă o înregistrare nouă în DNS pentru domeniul tău:

    SELECTOR._domainkey     TXT     "v=DKIM1; k=rsa; p=[cheia RSA publică]"

Dacă cheia **RSA** are mai mult de 1024-biți este nevoie de împărțirea înregistrării pe mai multe linii astfel:

    SELECTOR._domainkey     TXT     ( "v=DKIM1; k=rsa; p=MIIBIj..........."
                                    "....................................." )

Dacă știi sigur că toate mesajele din domeniul tău sunt semnate **DKIM** adaugă următoarele înregistrări în **DNS** pentru a declara că dorești rejectarea tuturor mesajelor false:

    _domainkey              TXT     "o=-; n=Toate mesajele autentice sunt semnate DKIM"
    _adsp._domainkey        TXT     "dkim=discardable"

### Verifică înregistrările DNS

    DOMAIN="Domeniul-Tău"
    SELECTOR="Un-Selector"
    dig +short $SELECTOR._domainkey.$DOMAIN TXT

    dig +short _domainkey.$DOMAIN TXT
    dig +short _adsp._domainkey.$DOMAIN TXT


Configurare serviciu
--------------------

Editează fișierul de configurare `/etc/opendkim.conf` să conțină:

    Syslog                  yes
    SyslogSuccess           yes
    #LogWhy                 yes
    UMask                   002

    Domain                  DOMAIN
    SubDomains              yes
    KeyFile                 /etc/mail/dkim_SELECTOR.key
    Selector                SELECTOR
    Canonicalization        relaxed/relaxed
    OversignHeaders         From

    #InternalHosts          /etc/mail/dkim_internalhosts
    On-BadSignature         tempfail
    On-DNSError             accept
    #On-Default             accept
    #RemoveARAll            yes
    #RemoveOldSignatures    yes
    Socket                  inet:8891@[127.0.0.1]

    MilterDebug             3


Adaugă autentificare DKIM în Postfix
------------------------------------

Pentru verificarea mesajelor primite prin portul standard **SMTP** (25), editează fișierul principal de configurare `/etc/postfix/main.cf` să conțină:

    smtpd_milters = [..] inet:127.0.0.1:8891

Pentru semnarea mesajelor trimise prin portul standard **submission** (587), editează fișierul de configurare `/etc/postfix/master.cf` să conțină:

    submission inet n       -       -       -       -       smtpd
      [..]
      -o smtpd_milters=[..],inet:127.0.0.1:8891
      -o milter_macro_daemon_name=ORIGINATING

Observație: filtrul **DKIM** trebuie procesat întotdeauna ultimul pentru a nu invalida semnătura de autenticitate.


Fapte cunoscute
---------------

### E: named[PID#]: dns_rdata_fromtext: „DOMAIN_file:LINE#” ran out of space

Această problemă apare pentru înregistrări `TXT` foarte lungi.
Împarte șirul în secvențe de cel mult 255 caractere pe fiecare linie.
