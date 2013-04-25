postfix 2.9.6-2 (mail)
======================

[Postfix][acasă] — este un server email **SMTP** pentru sistemele Linux/UNIX.

[acasă]: http://www.postfix.org/


Instalare pachete
-----------------

    HOSTu:~# apt install postfix postfix-pcre pfqueue gnutls-bin ca-certificates
    The following extra packages will be installed:
      libpfqueue0 ssl-cert

    General type of mail configuration:  Internet Site

    System mail name:                    HOST.DOMAIN

Notă: **apt** este un alias pentru `apt-get --purge` dar poți utiliza `aptitude` la fel de bine.

Module suplimentare:

* livrare locală: `dovecot-lda`
* anti-virus: `clamav` cu `clamav-milter`
* anti-spam: `spamassassin` cu `spamass-milter`
* autentificare mesaje: `opendkim`, `opendmarc`
* securitate: `fail2ban`

### Setează permisiuni pentru conturile noi

Editează fișierul de configurare `/etc/adduser.conf` și modifică:

    LETTERHOMES=yes
    DIR_MODE=0711

Fără aceste permisiuni este posibil ca alți utilizatori locali să poată citi documente sau fișiere private din contul celorlalți utilizatori.


Configurare serviciu
--------------------

#### Setează domeniul email implicit

    echo "${DOMAIN:-$(dnsdomainname)}" > /etc/mailname

### Configurare de bază Postfix

Editează fișierul de configurare `/etc/postfix/main.cf` să conțină:

    smtpd_banner = $myhostname ESMTP $mail_name (Debian/GNU)
    biff = no

    append_dot_mydomain = no

    delay_warning_time = 20h
    readme_directory = no

    # TLS parameters
    #smtpd_use_tls=yes

    smtpd_tls_CAfile = /etc/ssl/certs/ca-certificates.crt
    smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt

    alias_maps = hash:/etc/aliases
    alias_database = hash:/etc/aliases
    mydomain = DOMAIN
    myhostname = smtp.DOMAIN
    myorigin = /etc/mailname
    mydestination = $mydomain $myhostname $myorigin localhost HOST.DOMAIN
    #relay_domains = <ALTE DOMENII>
    #relayhost =
    mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128

    home_mailbox = mail/
    #mailbox_command =
    mailbox_size_limit = 0
    message_size_limit = 40960000
    recipient_delimiter = +
    inet_interfaces = all
    inet_protocols = ipv4
    smtp_bind_address = <IP_addr>

    smtpd_helo_required = yes
    smtpd_discard_ehlo_keywords = silent-discard dsn
    notify_classes = software resource delay 2bounce
    strict_rfc821_envelopes = yes

    # source IP address
    smtpd_client_restrictions =
      permit_inet_interfaces
      permit_mynetworks
      permit_sasl_authenticated
      reject_unknown_reverse_client_hostname
      #reject_unknown_client_hostname

    # MAIL FROM:
    smtpd_sender_restrictions =
      reject_non_fqdn_sender
      #hash:/etc/postfix/smtpd_sender_restrictions

    # RCPT TO:
    smtpd_recipient_restrictions =
      reject_non_fqdn_recipient
      reject_unknown_recipient_domain
      permit_inet_interfaces
      #permit_mynetworks               # open relay
      permit_sasl_authenticated
      reject_unauth_destination
      #reject_unverified_recipient
    #unknown_local_recipient_reject_code = 450

    # DATA
    smtpd_data_restrictions =
      reject_unauth_pipelining

    #debug_peer_list = <IP_addr/HOST/NET>
    #soft_bounce = yes

Activarea restricției `reject_unverified_recipient` este necesară doar pe **MX**-ul secundar.

Observații:

* unknown_client_reject_code (implicit: 450)
* non_fqdn_reject_code       (implicit: 504)

### Utilizează *Dovecot LDA* pentru livrarea locală a mesajelor

Editează fișierul de configurare `/etc/postfix/main.cf` să conțină:

    mailbox_command = /usr/lib/dovecot/dovecot-lda -f "$SENDER" -a "$RECIPIENT"

### Setează restricții pe baza antetelor *MIME*

O simplă restricție este blocarea mesajelor ce conțin fișiere atașate cu anumite extensii.

Editează fișierul de configurare `/etc/postfix/main.cf` să conțină:

    mime_header_checks = pcre:/etc/postfix/mime_header_checks.pcre

Editează fișierul text `/etc/postfix/mime_header_checks.pcre` să conțină:

    /^Content-(Disposition|Type).*name\s*=\s*"?[^>]*\.(bat|c[ho]m|cmd|dll|exe|lnk|pif|scr|vb[esx])"?\s*(;|$)/      REJECT

Un filtru mai eficient este menționat în pagina de manual `header_checks(5)`:

    /^Content-(Disposition|Type).*name\s*=\s*"?(.*(\.|=2E)(
      ade|adp|asp|bas|bat|chm|cmd|com|cpl|crt|dll|exe|
      hlp|ht[at]|
      inf|ins|isp|jse?|lnk|md[betw]|ms[cipt]|nws|
      \{[[:xdigit:]]{8}(?:-[[:xdigit:]]{4}){3}-[[:xdigit:]]{12}\}|
      ops|pcd|pif|prf|reg|sc[frt]|sh[bsm]|swf|
      vb[esx]?|vxd|ws[cfh]))(\?=)?"?\s*(;|$)/x
        REJECT Attachment name "$2" may not end with ".$4"

### Setează tabela de transport către anumite domenii

Editează fișierul text `/etc/postfix/transport_maps` să conțină:

    DOMAIN1.tld     smtp:[smtp.DOMAIN.tld]
    DOMAIN2.tld     smtp:[mail.DOMAIN.tld]
    DOMAIN3.tld     smtp:DOMAIN.tld

Notă: utilizează forma `[hostname]` pentru a livra direct către înregistrarea **A**, iar forma fără paranteze pentru a livra către înregistrarea **MX** a domeniului specificat.

Editează fișierul de configurare `/etc/postfix/main.cf` să conțină:

    transport_maps = hash:/etc/postfix/transport_maps

Execută comanda de actualizare și activare a tabelei de transport:

    postmap /etc/postfix/transport_maps
    service postfix reload

### Setează tabela de restricții de la anumite domenii

Editează fișierul text `/etc/postfix/smtpd_sender_restrictions` să conțină:

    DOMAIN1.tld     reject_unverified_sender
    DOMAIN2.tld     reject_unverified_sender

Editează fișierul de configurare `/etc/postfix/main.cf` să conțină:

    smtpd_sender_restrictions =
      [..]
      hash:/etc/postfix/smtpd_sender_restrictions

Execută comanda de actualizare și activare a tabelei de transport:

    postmap /etc/postfix/smtpd_sender_restrictions
    service postfix reload

### Setează restricții de la anumite adrese IP (RBL)

Editează fișierul de configurare `/etc/postfix/main.cf` să conțină:

    smtpd_client_restrictions =
      [..]
      reject_rbl_client bl.spamcop.net
      reject_rbl_client dnsbl-1.uceprotect.net
      reject_rbl_client dnsbl-2.uceprotect.net
      #reject_rbl_client dnsbl-3.uceprotect.net
      reject_rbl_client zen.spamhaus.org
      defer

Observații:

* maps_rbl_reject_code (default: 554)

### Inițializare «/etc/aliases»

    # /etc/aliases
    mailer-daemon: postmaster
    double-bounce: postmaster
    postmaster: root
    hostmaster: root
    webmaster: root
    ftpmaster: root
    abuse: root
    security: root

    root: <sysadmins>
    #administrator: root

O listă opțională de adrese generice des întâlnite:

    #---- adrese uzuale ----
    www: root
    ftp: root
    noc: root
    list: root

    #---- cont servicii ----
    archiva: root
    bacula: root
    bind: root
    changetrack: root
    clamav: root
    cvsd: root
    cvs-data: root
    daemon: root
    dovecot: root
    dk: root
    hg: root
    jira: root
    nagios: root
    nobody: root
    opendkim: root
    opendmarc: root
    postfix: root
    postgres: root
    snmp: root
    spamass-milter: root
    wpadmin: root
    www-data: root
    zabbix: root

    noreply: /dev/null
    no-reply: /dev/null
    nu-raspunde: /dev/null


Activare comunicare securizată TLS
----------------------------------

Editează fișierul de configurare `/etc/postfix/main.cf` să conțină:

    smtpd_tls_cert_file = /etc/ssl/certs/smtp.DOMAIN.crt
    smtpd_tls_key_file = /etc/ssl/private/smtp.DOMAIN.key
    #smtpd_use_tls=yes
    smtpd_tls_session_cache_database = btree:${data_directory}/smtpd_scache
    smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache

    smtpd_tls_CAfile = /etc/ssl/certs/ca-certificates.crt
    smtp_tls_CAfile  = /etc/ssl/certs/ca-certificates.crt

    # TLS outgoing connections (client-side engine)
    smtp_tls_note_starttls_offer = yes
    smtp_tls_security_level = may
    smtp_tls_loglevel = 1

    # TLS incomming connections (server-side engine)
    smtpd_tls_ask_ccert = yes
    smtpd_tls_security_level = may
    smtpd_tls_loglevel = 1
    smtpd_tls_received_header = yes

Pentru a putea citii certificatele **SSL**, adaugă contul în grupul `ssl-cert`:

    HOSTu:~# addgroup postfix ssl-cert


Activare autentificare conturi cu *SASL2*
-----------------------------------------

Cerințe:

* activat comunicare securizată **TLS**
* instalare și configurare serviciu `saslauthd`

### Setează metoda de verificare parolă

*Cyrus SASL* are un fișier de configurare pentru fiecare aplicație.

    HOSTu:~# cat > /etc/postfix/sasl/smtpd.conf <<__EOF__
    pwcheck_method: saslauthd
    mech_list: PLAIN LOGIN
    __EOF__

Lista completă a mecanismelor de autentificare **SASL** suportate se poate obține astfel:

    HOSTu:~# saslauthd -v
    saslauthd 2.1.25
    authentication mechanisms: sasldb getpwent kerberos5 pam rimap shadow ldap

#### Doar conturile din grupul «sasl» se pot conecta la priza SASL

    HOSTu:~# addgroup postfix sasl

### Autentificare obligatorie pe portul 587 (submission)

Editează fișierul `/etc/postfix/master.cf` să conțină:

    # ==========================================================================
    # service type  private unpriv  chroot  wakeup  maxproc command + args
    #               (yes)   (yes)   (yes)   (never) (100)
    # ==========================================================================
    submission inet n       -       -       -       -       smtpd
      -o syslog_name=postfix/submission
      -o smtpd_tls_security_level=encrypt
      -o smtpd_sasl_auth_enable=yes
      -o smtpd_sasl_authenticated_header=yes
      -o smtpd_client_restrictions=permit_sasl_authenticated,reject
      -o milter_macro_daemon_name=ORIGINATING

Comunicația pe portul standard `submission` se face doar prin activare **TLS**.
După autentificarea **SASL** se permite trimiterea de mesaje către orice destinație.

Poți configura `reject_unauth_destination` pentru a recepționa mesajele către domeniile locale fără autentificare — la fel ca portul standard **SMTP** (25).


### Autentificare opțională pe portul 25 (smtp)

Editează fișierul `/etc/postfix/main.cf` să conțină:

    # auth SASL in plain text on TCP port #25
    smtpd_sasl_auth_enable = yes
    smtpd_sasl_authenticated_header = yes
    #smtpd_sasl_security_options = noplaintext, noanonymous
    #smtpd_sasl_tls_security_options = noanonymous

    # force auth over TLS
    smtpd_tls_auth_only = yes

    # RCPT TO:
    smtpd_recipient_restrictions =
      # [..]
      permit_sasl_authenticated
      reject_unauth_destination


Fapte cunoscute
---------------

### E: postfix/local: [..] dsn=5.4.6, status=bounced (mail forwarding loop for USER@DOMAIN.tld)

Acesta este un mecanism intern `local(8)` pentru a prevenii un ciclu infinit de redirectări. În implementare este verificată prezența unui câmp `Delivered-To: USER@DOMAIN.tld`.

Pentru a corecta aceste posibile (dar rare) cicluri de redirectare, poți amâna răspunsul către expeditor prin editarea fișierului de configurare `/etc/postfix/master.cf` să conțină:

    local     unix  -       n       n       -       -       local
      -o soft_bounce=yes

După activarea acestei configurații, mesajele sunt puse în coadă și în jurnal vei găsi un cod de eroare temporară la livrare:

    postfix/local: [..] dsn=4.4.6, status=SOFTBOUNCE (mail forwarding loop for USER@DOMAIN.tld)

### W: postfix/smtpd[PID]: lost connection after UNKNOWN/CONNECT

Contextul complet din jurnal arată astfel:

    DATE HOST postfix/submission/smtpd[PID]: connect from REMOTEHOST[REMOTEIPADDR]
    DATE HOST postfix/submission/smtpd[PID]: lost connection after UNKNOWN
                                               from REMOTEHOST[REMOTEIPADDR]
    DATE HOST postfix/submission/smtpd[PID]: disconnect from REMOTEHOST[REMOTEIPADDR]

    DATE HOST postfix/submission/smtpd[PID]: connect from REMOTEHOST[REMOTEIPADDR]
    DATE HOST postfix/submission/smtpd[PID]: lost connection after CONNECT
                                               from REMOTEHOST[REMOTEIPADDR]
    DATE HOST postfix/submission/smtpd[PID]: disconnect from REMOTEHOST[REMOTEIPADDR]

    DATE HOST postfix/submission/smtpd[PID]: connect from REMOTEHOST[REMOTEIPADDR]
    DATE HOST postfix/submission/smtpd[PID]: Anonymous TLS connection established
                                               from REMOTEHOST[REMOTEIPADDR]: TLSv1
                                               with cipher RC4-MD5 (128/128 bits)

O posibilă cauză este inițierea unei conexiuni **SSL** de către clientul email (ex. *M.Outlook 2003*), iar în caz de eșec este inițiată conexiunea **TLS**.
