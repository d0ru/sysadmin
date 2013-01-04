spamassassin 3.3.2-4 (mail)
===========================

[Apache SpamAssassin][acasă] — un anti-spam pentru Linux/UNIX.

[acasă]: http://spamassassin.apache.org/


Instalare pachete
-----------------

    HOSTu:~# apt install spamassassin
    The following extra packages will be installed:
      binutils cpp cpp-4.7 gcc gcc-4.7 libc-dev-bin libc6-dev liberror-perl
      libgomp1 libio-socket-inet6-perl libitm1 libmail-spf-perl libmpc2 libmpfr4
      libnetaddr-ip-perl libquadmath0 libsocket6-perl libsys-hostname-long-perl
      linux-libc-dev make manpages-dev re2c spamc

    HOSTu:~# apt install libmail-dkim-perl pyzor razor
    The following extra packages will be installed:
      libcrypt-openssl-bignum-perl libcrypt-openssl-rsa-perl python-gdbm
      python-support

Notă: **apt** este un alias pentru `apt-get --purge` dar poți utiliza `aptitude` la fel de bine.

### Adaugă acces în „firewall”

    iptables -A FORWARD -d public.pyzor.org -p UDP --dport 24441 -j ACCEPT
    iptables -A FORWARD -d discovery.razor.cloudmark.com -p TCP --dport 2703 -j ACCEPT


Configurare serviciu
--------------------

#### :: /etc/spamassassin/v310.pre

    loadplugin Mail::SpamAssassin::Plugin::Pyzor
    loadplugin Mail::SpamAssassin::Plugin::Razor2
    loadplugin Mail::SpamAssassin::Plugin::AutoLearnThreshold
    loadplugin Mail::SpamAssassin::Plugin::WhiteListSubject

    #loadplugin Mail::SpamAssassin::Plugin::SpamCop
    loadplugin Mail::SpamAssassin::Plugin::MIMEHeader
    loadplugin Mail::SpamAssassin::Plugin::ReplaceTags

Notă: doar `MIMEHeader` și `ReplaceTags` nu sunt implicit activate.

#### :: /etc/spamassassin/v320.pre

    loadplugin Mail::SpamAssassin::Plugin::Check
    loadplugin Mail::SpamAssassin::Plugin::HTTPSMismatch
    loadplugin Mail::SpamAssassin::Plugin::URIDetail

    loadplugin Mail::SpamAssassin::Plugin::Bayes
    loadplugin Mail::SpamAssassin::Plugin::BodyEval
    loadplugin Mail::SpamAssassin::Plugin::DNSEval
    loadplugin Mail::SpamAssassin::Plugin::HTMLEval
    loadplugin Mail::SpamAssassin::Plugin::HeaderEval
    loadplugin Mail::SpamAssassin::Plugin::MIMEEval
    loadplugin Mail::SpamAssassin::Plugin::RelayEval
    loadplugin Mail::SpamAssassin::Plugin::URIEval
    loadplugin Mail::SpamAssassin::Plugin::WLBLEval

    loadplugin Mail::SpamAssassin::Plugin::VBounce
    loadplugin Mail::SpamAssassin::Plugin::Rule2XSBody
    loadplugin Mail::SpamAssassin::Plugin::ImageInfo

Notă: doar `Rule2XSBody` nu este implicit activat.

#### :: /etc/spamassassin/local.cf

    report_safe 0
    trusted_networks <INTRANET>

    lock_method flock
    required_score 6.00

    bayes_ignore_header X-Bogosity
    bayes_ignore_header X-Spam-Flag
    bayes_ignore_header X-Spam-Status
    bayes_ignore_header X-Spam-Checker-Version
    bayes_ignore_header X-Spam-Level
    bayes_ignore_header X-Spam-Score
    bayes_ignore_header X-Spam-Report
    bayes_ignore_header X-Virus-Scanned
    bayes_ignore_header X-Virus-Status
    bayes_ignore_header Z-Spam-Flag
    bayes_ignore_header Z-Spam-Score
    bayes_ignore_header Z-Spam-Status

    clear_headers  #clear the list of default headers to be added to messages
    add_header all Flag _YESNOCAPS_
    add_header all Status _YESNO_, score=_SCORE_ required=_REQD_ bayes=_BAYES_ autolearn=_AUTOLEARN_ tests=[_TESTSSCORES(,)_]
    add_header spam Level _STARS(*)_

    whitelist_from_rcvd  *@*DOMAIN1  DOMAIN1
    whitelist_from_rcvd  *@*DOMAIN2  DOMAIN2
    whitelist_from_rcvd  USER@DOMAIN  HOSTING-SITE

### Activează serviciul „anti-spam”

Editează fișierul de configurare `/etc/default/spamassassin` să conțină:

    ENABLED=1
    NICE="--nicelevel 10"
    CRON=1

### Pornește serviciul „anti-spam”

    HOSTu:~# service spamassassin start
