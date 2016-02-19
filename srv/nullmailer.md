nullmailer :: 1.11 (mail)
=========================

[nullmailer][acasă] — un **MTA** simplu pentru livrarea mesajelor de pe sistemul de calcul local către un sistem „e-mail” dedicat numit *smarthost*.

[acasă]: http://untroubled.org/nullmailer/


Instalare pachete
-----------------

    HOSTu:~# apt install nullmailer

    Mailname of your system:                <HOST.DOMENIU>
    Smarthosts:                             mail.<DOMENIU>
    Where to send local emails (optional):  root@<DOMENIU>

Numele sistemului este preluat din locația standard `/etc/mailname`.


Configurare serviciu
--------------------

### La ce adresă sunt livrate toate mesajele?

    echo "root+$(hostname -s)@${DOMENIU:-$(dnsdomainname)}" > /etc/nullmailer/adminaddr

Variabila „$DOMENIU” poate fi folosită pentru o destinație ce nu are legătură cu domeniul local.

    DOMENIU="nume-domeniu-extern.TLD"

Atenție: **toate** mesajele vor fi livrate la adresa specificată.

#### Activează comunicația securizată prin „TLS”

    HOSTu:~# dpkg-reconfigure nullmailer

    Mailname of your system:  <HOST.DOMENIU>
    Smarthosts:               mail.<DOMENIU> smtp --starttls --port 587

Notă: în versiunea 1.11 a fost adăugat codul pentru conexiuni securizate SSL/TLS.

#### Setează timpul de retrimitere

Implicit, `nullmailer` reîncearcă trimiterea mesajelor la fiecare minut.
Poți configura la ce interval să încerce trimiterea mesajelor din coadă prin editarea fișierului `/etc/nullmailer/pausetime` (ex. `1200` pentru un interval de 20 minute).


Fapte cunoscute
---------------

### Ignoră erorile *SMTP* permanente (cod 5.x.y)

Pentru că [nu șterge][5XY] mesajele din coadă, `nullmailer` încearcă periodic să retrimită mesaje ce sunt respinse de serverul *SMTP*. Singura soluție este ștergerea manuală a fișierelor din directorul `/var/spool/nullmailer/queue/`.

[5XY]: http://bugs.debian.org/329192

### „adminaddr” nu permite livrarea mesajelor către adresa personală a fiecărui cont

Dacă este configurat `adminaddr` nu se mai pot trimite mesaje (prin Cron) la adrese personale ale utilizatorilor (ID > 1000). Un exemplu de notificare per cont este mecanismul „commit watch” al serviciului *CVS*.
