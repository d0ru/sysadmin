clamav-milter 0.97.6+dfsg-1 (utils)
===================================

[ClamAV][acasă]-milter — verificare anti-virus în serviciul **MTA** (ex. postfix).

[acasă]: http://www.clamav.net/


Instalare pachete
-----------------

Cerință pre-instalare: `clamav`

    HOSTu:~# apt install clamav-milter
    The following extra packages will be installed:
      libmilter1.0.1

Notă: **apt** este un alias pentru `apt-get --purge` dar poți utiliza `aptitude` la fel de bine.


Configurare serviciu
--------------------

### Setează opțiuni prin reconfigurare pachet

    HOSTu:~# dpkg-reconfigure clamav-milter

    Communication interface with Sendmail:   inet:7357@127.0.0.1

    Action to perform on infected messages:  REJECT
     - Reject    : immediately refuse delivery (with a 5xx error);
     - Defer     : return a temporary failure message (4xx);
     - Blackhole : accept the message then drop it;
     - Quarantine: accept the message then quarantine it.

    Action to perform on error conditions:   DEFER	-- default

    Specific rejection reason for infected messages:
      "Service unavailable; Blocked INFECTED (%v)"

    Add headers to processed messages?       No

    Type of syslog messages:                 LOG_MAIL
    Enable verbose logging?                  Yes
    Information to log on infected messages: Full

    Size limit for scanned messages (MB):    60M


Adaugă protecție anti-virus în Postfix
--------------------------------------

Pentru scanarea mesajelor primite prin portul standard **SMTP** (25), editează fișierul principal de configurare `/etc/postfix/main.cf` să conțină:

    smtpd_milters = inet:127.0.0.1:7357

Pentru scanarea mesajelor trimise/primite prin portul standard **submission** (587), editează fișierul de configurare `/etc/postfix/master.cf` să conțină:

    submission inet n       -       -       -       -       smtpd
      [..]
      -o smtpd_milters=inet:127.0.0.1:7357
      -o milter_macro_daemon_name=ORIGINATING
