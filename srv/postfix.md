postfix 2.9.3-2.1 (mail)
========================

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

* livrare locală: `dovecot-lda` sau `procmail`
* anti-virus: `clamav` cu `clamav-milter`
* anti-spam: `spamassassin` cu `spamass-milter`
* autentificare mesaje: `opendkim`, `opendmarc`

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

    mydomain = DOMAIN
    myhostname = smtp.DOMAIN
    alias_maps = hash:/etc/aliases
    alias_database = hash:/etc/aliases

    myorigin = /etc/mailname
    mydestination = $mydomain $myhostname $myorigin localhost HOST.DOMAIN
    #relay_domains = <ALTE DOMENII>
    #relayhost =
    mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128
    #mailbox_command =
    home_mailbox = mail/
    mailbox_size_limit = 0
    message_size_limit = 40960000
    recipient_delimiter = +
    inet_interfaces = all
    inet_protocols = ipv4

    #debug_peer_list = <IP_addr/HOST/NET>
    #soft_bounce = yes
    strict_rfc821_envelopes = yes
    smtpd_helo_required = yes

    mime_header_checks = pcre:/etc/postfix/mime_header_checks
    smtp_bind_address = <IP_addr>
    #transport_maps = hash:/etc/postfix/transport_maps

    # uncomment 'permit_mynetworks' to enable IPaddr based relay
    smtpd_recipient_restrictions =
      permit_inet_interfaces
      #permit_mynetworks
      reject_non_fqdn_recipient
      reject_unknown_recipient_domain
      permit_sasl_authenticated
      reject_unauth_destination
      #reject_unverified_recipient

    smtpd_sender_restrictions =
      permit_inet_interfaces
      reject_non_fqdn_sender
      reject_unknown_sender_domain
      #hash:/etc/postfix/smtpd_sender_restrictions
      permit

Activarea restricției `reject_unverified_recipient` este necesară doar pe **MX**-ul secundar.

### Setează restricții pe baza antetelor MIME

O simplă restricție este blocarea mesajelor ce conțin fișiere atașate cu anumite extensii.

    HOSTu:~# cat > /etc/postfix/mime_header_checks <<__EOF__
    /^\s*Content-(Disposition|Type).*name\s*=\s*[^>]*\.(bat|c[ho]m|cmd|cpl|exe|dll|lnk|pif|scr|vb[esx])"?$/  REJECT
    __EOF__

### Inițializare «/etc/aliases»

    # /etc/aliases
    mailer-daemon: postmaster
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
    changetrack: root
    wpadmin: root

    #---- cont servicii ----
    archiva: root
    bind: root
    clamav: root
    cvsd: root
    cvs-data: root
    daemon: root
    dovecot: root
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
    smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt

    # TLS outgoing connections (client-side engine)
    smtp_tls_note_starttls_offer = yes
    smtp_tls_security_level = may
    smtp_tls_loglevel = 1

    # TLS incomming connections (server-side engine)
    smtpd_tls_ask_ccert = yes
    smtpd_tls_security_level = may
    smtpd_tls_loglevel = 1
    smtpd_tls_received_header = yes

Pentru a putea citii certificatele **SSL**/**TLS**, adaugă contul în grupul `ssl-cert`:

    HOSTu:~# addgroup postfix ssl-cert


Activare autentificare conturi cu SASL2
---------------------------------------

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
      -o smtpd_client_restrictions=permit_sasl_authenticated,reject_unauth_destination
      -o milter_macro_daemon_name=ORIGINATING

Comunicația pe portul standard `submission` se face doar prin activare **TLS**.
După autentificarea **SASL** se permite trimiterea de mesaje către orice destinație.

Am configurat `reject_unauth_destination` pentru a recepționa mesajele către domeniile locale fără autentificare — la fel ca portul standard **SMTP** (25).


### Autentificare opțională pe portul 25 (smtp)

Editează fișierul `/etc/postfix/main.cf` să conțină:

    # auth SASL in plain text on TCP port #25
    smtpd_sasl_auth_enable = yes
    smtpd_sasl_authenticated_header = yes
    #smtpd_sasl_security_options = noplaintext, noanonymous
    #smtpd_sasl_tls_security_options = noanonymous

    # force auth over TLS
    smtpd_tls_auth_only = yes

    # [..]
    smtpd_recipient_restrictions =
      # [..]
      permit_sasl_authenticated
      reject_unauth_destination
