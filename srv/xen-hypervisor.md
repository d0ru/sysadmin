xen-hypervisor :: 4.0.1-5.2 (kernel)
====================================

[Xen][home] este o tehnologie avansată de virtualizare a sistemelor de calcul.
Componenta principală este hipervizorul (eng. „hypervisor”), însă la fel de importante sunt și uneltele pentru administrarea hosturilor virtuale (numite și „guests”).

[home]: http://www.xen.org/

O diferență esențială față de tehnologia **KVM** este că **Xen** funcționează cu procesoare fără suport de **virtualizare hardware**, furnizat de capabilitatea `SVM` la *AMD* sau `VT` la *Intel*.

    HOSTu:~# egrep --color=always '(svm|vmx)' /proc/cpuinfo
    flags		: .. svm ..
      or
    flags		: .. vmx ..

După încărcarea hipervizorului Xen este posibil ca această capabilitate să fie ascunsă.
Poți verifica suportul CPU pentru virtualizare astfel:

    HOSTu:~# xm dmesg | egrep --color=always -i '(svm|vmx)'
    (XEN) VMX: Supported advanced features:
    (XEN) HVM: VMX enabled


Instalare pachete
-----------------

### Construirea unui pod rețea pentru hosturile virtuale

    HOSTu:~# apt install bridge-utils

Editează fișierul `/etc/network/interfaces` ca să conțină:

    allow-hotplug eth0
    auto br0
    iface br0 inet static
      bridge_ports eth0
      address A.A.A.A
      netmask N.N.N.N

Notă: este necesară repornirea sistemului sau doar a rețelei (dacă știi ce faci).

### Hipervizorul Xen

Este nevoie de un kernel Linux cu suport pentru Xen DOM0. Orice kernel Linux ≥ 3.2 are inclus acest suport nativ.

    HOSTu:~# apt install linux-image-2.6-xen-amd64
    The following extra packages will be installed:
      linux-image-2.6.32-5-xen-amd64

Notă: **apt** este un alias pentru `apt-get --purge` dar poți utiliza `aptitude` la fel de bine.

Kernelul Linux DOM0 este încărcat de hipervizorul Xen.

    HOSTu:~# apt install xen-hypervisor
    Note, selecting 'xen-hypervisor-4.0-amd64' instead of 'xen-hypervisor'
    The following extra packages will be installed:
      gawk libxenstore3.0 python2.5 python2.5-minimal
      xen-hypervisor-4.0-amd64
      xen-utils-4.0 xen-utils-common xenstore-utils

    HOSTu:~# apt install xen-qemu-dm
    Note, selecting 'xen-qemu-dm-4.0' instead of 'xen-qemu-dm'
    The following extra packages will be installed:
      ca-certificates dbus esound-common etherboot etherboot-qemu libasound2
      libasyncns0 libaudiofile0 libbluetooth3 libbrlapi0.5 libcurl3-gnutls
      libdbus-1-3 libdirectfb-1.2-9 libdrm-intel1 libdrm-radeon1 libdrm2 libesd0
      libflac8 libgl1-mesa-dri libgl1-mesa-glx libice6 libogg0 libpulse0
      libsdl1.2debian libsdl1.2debian-alsa libsm6 libsndfile1 libsvga1 libsysfs2
      libts-0.0-0 libvde0 libvdeplug2 libvorbis0a libvorbisenc2 libx86-1
      libxdamage1 libxfixes3 libxi6 libxtst6 libxxf86vm1 mknbi openbios-ppc
      openbios-sparc openhackware openssl qemu-keymaps qemu-system qemu-utils
      seabios tsconf vde2 vgabios x11-common xen-qemu-dm-4.0

Este necesară reconfigurarea `grub2` pentru a încărca automat hipervizorul Xen la „boot”.
Editează fișierul `/etc/default/grub` ca să conțină:

    GRUB_CMDLINE_LINUX_DEFAULT="quiet"
    GRUB_CMDLINE_LINUX="panic=1200 nomodeset console=ttyS1,57600n8"
    GRUB_CMDLINE_XEN="dom0_mem=2048M"

Parametrul `console` este necesar doar pentru sistemele de calcul „headless” — fără echipamente periferice de consolă (monitor, tastatură).

Schimbă sistemul de operare încărcat implicit la pornire:

    sed -i "s%^\(GRUB_DEFAULT\)=.*$%\1=8%" /etc/default/grub
    update-grub

În cazul acesta kernelul *Linux 2.6.32* cu *XEN 4.0* se află pe poziția nr. 8 (cu numerotare de la zero).
Această poziție se poate determina astfel:

    grep ^menuentry /boot/grub/grub.cfg | awk -F\' '{
      if ($2=="Debian GNU/Linux, with Linux 2.6.32-5-xen-amd64 and XEN 4.0-amd64")
      print NR-1 }'

Înainte de repornirea sistemului de calcul nu uita să activezi „cpuvt” din BIOS.


Configurare serviciu
--------------------

#### Dezactivare scripturi de rețea pentru hosturile virtuale

    HOSTu:~# grep -v '^\s*\(#.*\)\?$' /etc/xen/xend-config.sxp
    sed "s|^\((network-script .*)\)$|#\1|" -i /etc/xen/xend-config.sxp

Nu este nevoie de execuția nici unui script pentru că hosturile virtuale vor folosi doar podul independent `br0` creat anterior.

#### Activare priza de conectare sau serverul HTTP

    sed "s|^#*\((xend-unix-server\) .*$|\1 yes)|" -i /etc/xen/xend-config.sxp
    sed "s|^#*\((xend-http-server\) .*$|\1 yes)|" -i /etc/xen/xend-config.sxp

Această priză este necesară pentru conectare cu «libvirtd», implicit și pentru `virsh`.

Doar unul din cele două este suficient, „unix” este mai sigur și elimină un posibil conflict pentru portul TCP 8000.

#### Creare spațiu de stocare a hosturilor virtuale

    mkdir -vp /srv/virt/images
    chmod -v 0700 /srv/virt/images

Este recomandat ca `/srv/virt` să fie o partiție dedicată hosturilor virtuale.

#### Consola VNC ar trebui să accepte conexiuni TCP de oriunde

    sed "s|^#\((vnc-listen\) .*$|\1 '0.0.0.0')|" -i /etc/xen/xend-config.sxp

Acest parametru face posibilă conectarea la consola sistem a hosturilor virtuale prin VNC.

#### Dezactivează salvarea stării hosturilor virtuale

    sed 's|^\(XENDOMAINS_SAVE\)=.*$|\1=""|' -i /etc/default/xendomains

Un parametru fără valoare va dezactiva salvarea hosturilor virtuale la oprirea/repornirea hipervizorului Xen. Efectul dorit este oprirea hosturilor virtuale înainte de către «xend».

#### Setează timpul de așteptare la 20 secunde

    sed 's|^\(XENDOMAINS_STOP_MAXWAIT\)=.*$|\1=20|' -i /etc/default/xendomains

Acesta este timpul de așteptare pentru fiecare host virtual ca să execute operația `shutdown`.


Opțional poți instala și configura serviciul `libvirtd` pentru o administrare a mașinilor virtuale folosind unelte independente de tehnologia de virtualizare (ex. *Xen*) cum ar fi `virsh`.
