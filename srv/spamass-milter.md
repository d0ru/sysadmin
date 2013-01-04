spamass-milter 0.3.2-1 (mail)
=============================

[SpamAssassin Milter][acasă] — verificare anti-spam în serviciul **MTA** (ex. postfix).

[acasă]: http://savannah.nongnu.org/projects/spamass-milt/


Instalare pachete
-----------------

Cerință pre-instalare: `spamassassin`

    HOSTu:~# apt install spamass-milter
    The following extra packages will be installed:
      libmilter1.0.1

Notă: **apt** este un alias pentru `apt-get --purge` dar poți utiliza `aptitude` la fel de bine.

#### Crează grup sistem «spamass-milter»

    addgroup --system spamass-milter
    usermod --gid spamass-milter spamass-milter

#### Crează director acasă pentru «spamass-milter»

    install -v -o spamass-milter -g spamass-milter -d /var/lib/spamass-milter


Configurare serviciu
--------------------

### Crează directorul de lucru

    install -v -o spamass-milter -g spamass-milter -m 0775 -d /var/spool/postfix/run/spamass-milter

### Setează parametrii de pornire

Editează fișierul `/etc/default/spamass-milter` să conțină:

    OPTIONS="-I -i 127.0.0.1,IP_addr,SUBNET -m -r 20 -- --headers"

    SOCKET="/var/spool/postfix/run/spamass-milter/mux"
    SOCKETOWNER="spamass-milter:spamass-milter"

* `-I` ignoră mesajul dacă utilizatorul a fost autentificat via „SMTP AUTH”  
* `-i networks` ignoră mesajul dacă adresa IP sursă este în rețelele listate  
* `-m` dezactivează modificarea câmpurilor `Subject:`, `Content-Type:` sau conținutul mesajului  
* `-r NN` rejectează mesajul scanat cu un scor de cel puțin `NN`  
* `-- --headers` parametru pentru `spamc` ce previne modificarea conținutului mesajului


Adaugă protecție anti-spam în Postfix
-------------------------------------

Editează fișierul de configurare `/etc/postfix/main.cf` să conțină:

    milter_connect_macros = j {daemon_name} v {if_name} _
    smtpd_milters = [..] unix:/run/spamass-milter/mux

Pentru a înțelege semnificația acestor macrouri citește [MILTER_README][milter].

[milter]: http://www.postfix.org/MILTER_README.html

Adaugă contul `postfix` în grupul `spamass-milter`.

    HOSTu:~# addgroup postfix spamass-milter


Fapte cunoscute
---------------

### W: Could not retrieve sendmail macro "i" — confMILTER_MACROS_ENVFROM

Mesajul complet este:

    Dec 28 02:42:06 HOSTNAME spamass-milter[PID]: Could not retrieve
      sendmail macro "i"!.  Please add it to confMILTER_MACROS_ENVFROM
      for better spamassassin results

[Problema][696856] este că aplicațiile *milter* așteaptă ca „queue ID” să fie cunoscut înainte ca **MTA** (ex. postfix) să accepte comanda `MAIL FROM`.

[696856]: http://bugs.debian.org/696856
