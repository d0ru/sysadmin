dovecot 1:2.1.7-6 (mail)
========================

[Dovecot][acasă] — este un server email **IMAP** și **POP3** pentru sistemele Linux/UNIX.

[acasă]: http://dovecot.org/


Instalare pachete
-----------------

    HOSTu:~# apt install dovecot-imapd dovecot-pop3d dovecot-managesieved
    The following extra packages will be installed:
      dovecot-core dovecot-sieve

Notă: **apt** este un alias pentru `apt-get --purge` dar poți utiliza `aptitude` la fel de bine.


Configurare serviciu
--------------------

În Debian serviciile *Dovecot* (ex. **IMAP** și *ManageSieve*) sunt implicit activate după instalare. Alte configurări suplimentare:

#### :: /etc/dovecot/dovecot.conf

    verbose_proctitle = yes
    shutdown_clients = yes

    !include conf.d/*.conf
    !include_try local.conf

#### :: /etc/dovecot/local.conf

    mail_location = maildir:~/mail

#### :: /etc/dovecot/conf.d/15-mailboxes.conf

    namespace inbox {
      mailbox Archives {
        auto = subscribe
        special_use = \Archive
      }
      mailbox Drafts {
        auto = subscribe
        special_use = \Drafts
      }
      mailbox Junk {
        auto = subscribe
        special_use = \Junk
      }
      mailbox Trash {
        auto = subscribe
        special_use = \Trash
      }

      mailbox Sent {
        auto = subscribe
        special_use = \Sent
      }
      mailbox "Sent Messages" {
        special_use = \Sent
      }
    }

Notă: nu toate aplicațiile client email implementează [RFC 6154][rfc6154].

[rfc6154]: https://tools.ietf.org/html/rfc6154

#### :: /etc/dovecot/conf.d/10-logging.conf

    #log_path = syslog
    info_log_path = /var/log/mail.info
    #syslog_facility = mail

    auth_verbose = yes
    #auth_verbose_passwords = no

    #mail_debug = no
    #verbose_ssl = no

    deliver_log_format = from=%f msgid=%m: %$, size=%p

Setări implicite ce pot fi verificate cu `doveconf`:

    disable_plaintext_auth = yes
    mail_privileged_group =


Configurare serviciu Dovecot IMAP
---------------------------------

Fișierul de configurare specific protocolului **IMAP** este `/etc/dovecot/conf.d/20-imap.conf`.

### Setează adresele IP pentru conexiunile IMAP

Editează fișierul `/etc/dovecot/conf.d/10-master.conf` să conțină:

    service imap-login {
      inet_listener imap {
        address = 127.0.0.1
        #port = 143
      }
      inet_listener imaps {
        address = <local IP addr>
        #port = 993
        #ssl = yes
      }
    }

### Setează certificatul SSL pentru serviciul IMAP

Editează fișierul `/etc/dovecot/local.conf` să conțină:

    protocol imap {
      ssl_cert = </etc/ssl/certs/imap.DOMAIN.crt
      ssl_key  = </etc/ssl/private/imap.DOMAIN.key
    }


Configurare serviciu Dovecot POP3
---------------------------------

Fișierul de configurare specific protocolului **POP3** este `/etc/dovecot/conf.d/20-pop3.conf`.

### Setează adresele IP pentru conexiunile POP3

Editează fișierul `/etc/dovecot/conf.d/10-master.conf` să conțină:

    service pop3-login {
      inet_listener pop3 {
        address = 127.0.0.1
        #port = 110
      }
      inet_listener pop3s {
        address = <local IP addr>
        #port = 995
        #ssl = yes
      }
    }

### Setează certificatul SSL pentru serviciul POP3

Editează fișierul `/etc/dovecot/local.conf` să conțină:

    protocol pop3 {
      ssl_cert = </etc/ssl/certs/pop.DOMAIN.crt
      ssl_key  = </etc/ssl/private/pop.DOMAIN.key
    }


Configurare serviciu Dovecot LDA
--------------------------------

Spre deosebire de celelalte module *Dovecot*, nu există un proces unic `dovecot-lda` ci este executat la cerere (de către „postfix” sau „procmail”).

Deoarece `dovecot-lda` nu este pornit de către `root` există cel puțin două soluții:

* creezi un fișier `/var/log/dovecot/lda.info` în care pot scrie toți utilizatorii (mod `0662`)
* setezi o facilitate Syslog diferită pentru **LDA** (ex. local2).

Pentru a implementa prima soluție editează fișierul `/etc/dovecot/conf.d/15-lda.conf` să conțină:

    #postmaster_address = postmaster@DOMAIN
    #submission_host = smtp.DOMAIN
    lda_mailbox_autocreate = yes
    lda_mailbox_autosubscribe = yes

    protocol lda {
      info_log_path = /var/log/dovecot/lda.info
      [..]
    }

### Inițializează jurnalul operațiilor LDA

    install -v -o dovecot -g adm -m 0751 -d /var/log/dovecot
    
    touch /var/log/dovecot/lda.info
    chown -v dovecot:adm /var/log/dovecot/lda.info
    chmod -v 0662 /var/log/dovecot/lda.info

Pentru rotirea automată a jurnalului crează fișierul `/etc/logrotate.d/dovecot-lda` să conțină:

    /var/log/dovecot/lda.info {
      create 0662 dovecot adm
      compress
      delaycompress
      notifempty
      rotate 60
      weekly
    }

### Configurează Postfix să utilizeze LDA pentru livrarea locală a mesajelor

Editează fișierul `/etc/postfix/main.conf` să conțină:

    mailbox_command = /usr/lib/dovecot/dovecot-lda -e -f "$SENDER" -a "$RECIPIENT"

### Activează și configurează plugin-ul «sieve»

Editează fișierul `/etc/dovecot/conf.d/15-lda.conf` să conțină:

    protocol lda {
      [..]
      mail_plugins = $mail_plugins sieve
    }

Editează fișierul de configurare `/etc/dovecot/conf.d/90-sieve.conf` să conțină:

    plugin {
      sieve = ~/.dovecot.sieve
      sieve_dir = ~/mail/sieve

      #sieve_global_dir =
      sieve_before = /etc/dovecot/sieve/before.d
      sieve_after  = /etc/dovecot/sieve/after.d
      sieve_extensions = +vnd.dovecot.duplicate
    }

Observații:

* am utilizat `~/mail/sieve` doar pentru a face backup automat la filtrele personale
* vechile extensii `notify` și `imapflags` au fost înlocuite cu `enotify`, respectiv `imap4flags`
* am activat extensia `vnd.dovecot.duplicate` pentru eliminarea mesajelor duplicate ce ar putea fi livrate în „Inbox”.

#### :: /etc/dovecot/sieve/before.d/0junk.sieve

    require ["fileinto"];

    # rule:[Mută mesajele nesolicitate în „Junk”]
    if header :contains "X-Spam-Flag" "YES"
    {
            fileinto "Junk";
            stop;
    }

#### :: /etc/dovecot/sieve/after.d/9dedup.sieve

    require ["vnd.dovecot.duplicate", "fileinto"];

    # rule:[Mută mesajele duplicate în „Trash”]
    if duplicate
    {
            fileinto "Trash";
            stop;
    }
