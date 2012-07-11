ssmtp :: 2.64-4 (mail)
======================

[ssmtp][home] este un „MTA” simplu pentru livrarea mesajelor de pe sistemul de calcul local către un sistem „e-mail” dedicat.

[home]: http://packages.qa.debian.org/s/ssmtp.html


Configurare serviciu
--------------------

#### Cine primește toate mesajele pentru conturile sistem? (ID < 1000)

    sed "s%^root=.*$%root=root+$(hostname -s)@${DOMAIN:-$(dnsdomainname)}%" -i /etc/ssmtp/ssmtp.conf

Variabila „$DOMAIN” poate fi folosită pentru o destinație ce nu are legătură cu domeniul local.

    DOMAIN="nume-domeniu-extern.TLD"

#### Utilizează numele complet al sistemului (FQDN)

    sed "s|^#*\(hostname\)=.*$|\1=$(hostname -f)|" -i /etc/ssmtp/ssmtp.conf

Dacă sistemul este configurat cu un nume local (nu poate fi rezolvat de un serviciu DNS extern) este nevoie de suprascrierea domeniului sursă:

    REWRITEDOMAIN="nume-domeniu-public.TLD"
    sed "s|^#*\(rewriteDomain\)=.*$|\1=$REWRITEDOMAIN|" -i /etc/ssmtp/ssmtp.conf

#### Unde vor fi livrate direct mesajele?

    MAILHUB="mail.$(dnsdomainname)"
    sed "s%^mailhub=.*$%mailhub=$MAILHUB%" -i /etc/ssmtp/ssmtp.conf

Implicit, portul standard SMTP (25) este utilizat. Aceasta se poate schimba astfel:

    MAILHUB="smtp.nume-domeniu.TLD:587"

#### Activează comunicația securizată prin „TLS”

    cat >> /etc/ssmtp/ssmtp.conf <<__EOF__

    # use TLS to talk to the SMTP server
    UseSTARTTLS=yes
    __EOF__

#### Forțează adresa reală „From:”

    sed "s|^#*\(FromLineOverride\)=.*$|\1=NO|" -i /etc/ssmtp/ssmtp.conf
