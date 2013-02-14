tightvncserver 1.3.9-6.4 (x11)
==============================

[TightVNC][acasă] — este un server **VNC** pentru controlul sistemului de la distanță într-o sesiune grafică (*X11*).

[acasă]: http://www.tightvnc.com/

Scurt glosar:

* **VNC** — „Virtual Network Computing”


Instalare pachete
-----------------

Cerință pre-instalare: `xserver-xorg`

    HOSTu:~# apt install tightvncserver x11-apps xterm
    The following extra packages will be installed:
      libutempter0 libxfont1 libxkbfile1 x11-xserver-utils xbitmaps
      xfonts-base xfonts-encodings xfonts-utils

Notă: **apt** este un alias pentru `apt-get --purge` dar poți utiliza `aptitude` la fel de bine.


Configurare serviciu
--------------------

Editează scriptul de pornire `~/.vnc/xstartup` să conțină:

    #!/bin/sh

    [ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup
    [ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
    xsetroot -solid grey
    x-window-manager &
    x-session-manager &

În scriptul de pornire pot fi adăugate și alte aplicații ce vor fi pornite automat:

    x-terminal-emulator -geometry 106x24+10+10 -ls -title "$VNCDESKTOP Desktop" &

Pentru ca serviciul să fie pornit automat la *boot*, adaugă următoarea linie în scriptul `/etc/rc.local`:

    su - root -c "vncserver -geometry 1152x864 -depth 24"


Verificare și testare
---------------------

Pentru a porni un nou server **VNC** execută comanda:

    vncserver -geometry 1152x864 -depth 24

Numerotarea serverelor **VNC** începe de la `:1`.

Pentru a oprii un server **VNC** execută comanda:

    vncserver -kill :2
