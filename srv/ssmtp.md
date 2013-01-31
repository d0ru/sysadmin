ssmtp :: 2.64-7 (mail)
======================

`ssmtp` este un „MTA” simplu pentru livrarea mesajelor de pe sistemul de calcul local către un sistem „e-mail” dedicat numit *smarthost*.

Este foarte util pentru o infrastructură ce utilizează nume interne (ex. `HOST.DOMAIN.local`), astfel mesajele trimise trebuiesc rescrise cu `DOMAIN.tld` — un domeniu real.


Configurare serviciu
--------------------

#### Cine primește toate mesajele pentru conturile sistem? (ID < 1000)

    sed "s|^root=.*$|root=root+$(hostname -s)@${DOMAIN:-$(dnsdomainname)}|" -i /etc/ssmtp/ssmtp.conf

Variabila „$DOMAIN” poate fi folosită pentru o destinație ce nu are legătură cu domeniul local.

    DOMAIN="nume-domeniu-extern.tld"

#### Setează numele complet al sistemului

    sed "s|^#*\([Hh]ostname\)=.*$|\1=$(hostname -f)|" -i /etc/ssmtp/ssmtp.conf

Dacă sistemul este configurat cu un nume local (ex. `HOST.DOMAIN.local`), ce nu poate fi rezolvat de orice serviciu DNS extern, este nevoie de rescrierea domeniului sursă:

    REWRITEDOMAIN="nume-domeniu-public.tld"
    sed "s|^#*\(rewriteDomain\)=.*$|\1=$REWRITEDOMAIN|" -i /etc/ssmtp/ssmtp.conf

#### Unde vor fi livrate direct mesajele?

    MAILHUB="mail.$(dnsdomainname)"
    sed "s|^mailhub=.*$|mailhub=$MAILHUB|" -i /etc/ssmtp/ssmtp.conf

*MAILHUB* referă un sistem „e-mail” dedicat, în anumite contexte numit *smarthost*.

#### Activează comunicația securizată prin „TLS”

    cat >> /etc/ssmtp/ssmtp.conf <<__EOF__

    # use TLS to talk to the SMTP server
    UseSTARTTLS=yes
    __EOF__

Implicit este utilizat portul standard *SMTP* (25). Acesta se poate modifica astfel:

    MAILHUB="HOST.DOMAIN.tld:587"
    sed "s|^mailhub=.*$|mailhub=$MAILHUB|" -i /etc/ssmtp/ssmtp.conf

#### Forțează adresa reală „From:”

    sed "s|^#*\(FromLineOverride\)=.*$|\1=NO|" -i /etc/ssmtp/ssmtp.conf


Fapte cunoscute
---------------

### Nu are o coadă de stocare a mesajelor

Orice problemă de comunicare cu serverul *mailhub* duce la pierderea acelui mesaj, iar conținutul este adăugat la fișierul local `~/dead.letter`.

### Nu completează @HOST.DOMAIN în adresele email

Dacă vreunul din câmpurile unui mesaj are o adresă incompletă (de obicei „TO: root”), mesajul va fi livrat fără modificare către *mailhub*.
Unele servere **SMTP** pot rejecta aceste mesaje (nu sunt conforme *RFC821*) sau completa adresa cu numele propriu (nu cel așteptat al hostului sursă).
