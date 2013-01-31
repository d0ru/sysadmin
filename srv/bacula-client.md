bacula-client 5.2.6+dfsg-8 (admin)
==================================

[Bacula][acasă] — un serviciu pentru copierea (eng. „backup”) și restaurarea datelor.

[acasă]: http://bacula.org/


Instalare pachete
-----------------

    HOSTu:~# apt install bacula-fd
    The following extra packages will be installed:
      bacula-common libpython2.7

Notă: **apt** este un alias pentru `apt-get --purge` dar poți utiliza `aptitude` la fel de bine.

### Fixează rotația jurnalului

Pentru [rotirea automată][694046] a jurnalului editează fișierul `/etc/logrotate.d/bacula-common` să conțină:

    /var/log/bacula/log {
      create  0640 bacula adm
      compress
      delaycompress
      missingok
      notifempty
      rotate 12
      monthly
    }

[694046]: http://bugs.debian.org/694046


Configurare serviciu
--------------------

    BACULAMASTER="numehost"
    sed "s|$(hostname)-dir|${BACULAMASTER}-dir|" -i /etc/bacula/bacula-fd.conf
    sed "s|#*FDAddress|#FDAddress|" -i /etc/bacula/bacula-fd.conf

    sed "s|$(hostname)-mon|bacula-tray-monitor|" -i /etc/bacula/bacula-fd.conf
    sed "s|$(hostname)-fd|$(hostname -s)-fd|" -i /etc/bacula/bacula-fd.conf

Fișierul de configurare `/etc/bacula/bacula-fd.conf` ar trebui să conțină:

    #
    # Default  Bacula File Daemon Configuration file
    #

    # List Directors who are permitted to contact this File daemon
    Director {
      Name = <BACULAMASTER>-dir
      Password = "************"
    }

    # Restricted Director, used by tray-monitor to get the
    #   status of the file daemon
    Director {
      Name = bacula-tray-monitor
      Password = "*************"
      Monitor = yes
    }

    # "Global" File daemon configuration specifications
    FileDaemon {
      Name = <HOST>-fd
      FDport = 9102
      WorkingDirectory = /var/lib/bacula
      Pid Directory = /var/run/bacula
      Maximum Concurrent Jobs = 20
      #FDAddress = 127.0.0.1
    }

    # Send all messages except skipped files back to Director
    Messages {
      Name = Standard
      Append = "/var/log/bacula/log" = all, !skipped
      Director = <BACULAMASTER>-dir = all, !skipped, !restored
    }

Parametrii `Director:Name` și `Director:Password` trebuie să coincidă exact cu valorile setate în configurația serviciului *Bacula Director*.
Parametrul *FileDaemon:Name* nu pare să aibă vreo restricție.

În plus, am configurat să păstreze local o copie a jurnalului.

### Securitate: limitează privilegiile serviciului

Editează fișierul de configurare `/etc/default/bacula-fd` să conțină:

    ARGS="-u bacula -g bacula -k"

Aceasta previne modificarea oricărui fișier, păstrează doar permisiunea de citire/copiere. Altfel serviciul [poate suprascrie orice fișier][699149].

[699149]: http://bugs.debian.org/699149
