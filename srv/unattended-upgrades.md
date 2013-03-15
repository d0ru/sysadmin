unattended-upgrades 0.79.5 (admin)
==================================

`unattended-upgrade` este o unealtă pentru instalarea automată a noilor pachete apărute în repozitoriile configurate, în special pentru fixarea problemelor de securitate.


Instalare pachete
-----------------

    HOSTu:~# apt install unattended-upgrades apt-listbugs

Notă: **apt** este un alias pentru `apt-get --purge` dar poți utiliza `aptitude` la fel de bine.

`apt-listbugs` este util pentru cei care preferă distibuția „unstable” — oferă o protecție împotriva problemelor grave raportate imediat.
Efectul provocat este anularea actualizării pachetelor atunci când unul din pachetele ce urmează a fii instalate are raportat un gândac de o mare gravitate.

Pentru [rotirea automată][702323] **doar** a jurnalului scurt, editează fișierul `/etc/logrotate.d/unattended-upgrades` să conțină:

    /var/log/unattended-upgrades/unattended-upgrades.log
    /var/log/unattended-upgrades/unattended-upgrades-shutdown.log {
      rotate 6
      monthly
      compress
      missingok
      notifempty
    }

[702323]: http://bugs.debian.org/702323


Configurare serviciu
--------------------

### Activează actualizarea automată

Reconfigurăm pachetul pentru a activa modulul **APT** „Unattended-Upgrade”.

    HOSTu:~# dpkg-reconfigure unattended-upgrades

    Automatically download and install stable updates?  <Yes>

### Instalare automată pachete din repozitoriul Debian

Activează actualizarea automată din repozitoriul Debian „stable”:

    CFGFILE="/etc/apt/apt.conf.d/50unattended-upgrades"

    sed 's|^//\(\s*"o=Debian,a=stable";\)$|  \1|' -i $CFGFILE
    sed 's|^//\(\s*"o=Debian,a=stable-updates";\)$|  \1|' -i $CFGFILE
    sed 's|^//\(\s*"o=Debian,a=proposed-updates";\)$|  \1|' -i $CFGFILE

Fișierul de configurare `/etc/apt/apt.conf.d/50unattended-upgrades` ar trebui să conțină:

    Unattended-Upgrade::Origins-Pattern {
            "o=Debian,a=stable";
            "o=Debian,a=stable-updates";
            "o=Debian,a=proposed-updates";
            "origin=Debian,archive=stable,label=Debian-Security";
    };

Activează actualizarea automată din repozitoriul Debian „testing”:

    sed 's|^.*"o=Debian,a=stable";.*$|        "o=Debian,a=testing";\n&|' -i $CFGFILE

Activează actualizarea automată din repozitoriul Debian „unstable”:

    sed 's|^.*"o=Debian,a=stable";.*$|        "o=Debian,a=unstable";\n&|' -i $CFGFILE

### Instalare automată pachete din repozitoriul Google

Pentru actualizarea automată a pachetelor din repozitoriul Google (ex. google-chrome, google-talkplugin) editează fișierul de configurare `/etc/apt/apt.conf.d/50unattended-upgrades` să conțină:

    Unattended-Upgrade::Origins-Pattern {
            "o=Debian,a=stable";
            "o=Debian,a=stable-updates";
            "o=Debian,a=proposed-updates";
            "origin=Debian,archive=stable,label=Debian-Security";
            "origin=Google\, Inc.,archive=stable,label=Google";
    };

### Notificare prin email

Pentru ca serviciul să poată trimite mesaje este nevoie de instalarea unei aplicații client compatibilă `sendmail` (ex. nullmailer) sau `mailx` (ex, bsd-mailx).

Activează trimiterea unui mesaj de notificare prin email:

    CFGFILE="/etc/apt/apt.conf.d/50unattended-upgrades"

    sed 's|^/*\(Unattended-Upgrade::Mail\) .*$|\1 "root";|' -i $CFGFILE

Suplimentar, poți activa trimiterea unui mesaj **doar** în caz de eroare:

    sed 's|^/*\(Unattended-Upgrade::MailOnlyOnError\) .*$|\1 "true";|' -i $CFGFILE
