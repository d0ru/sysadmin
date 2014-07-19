Debian Linux 6.0.10 (nume de cod „squeeze”)
===========================================

Debian 6.0 a fost lansat inițial la 6 februarie, 2011.

Debian 6.0 a fost înlocuit de Debian 7 („wheezy”).


Instalare host
--------------

Comanda implicită de instalare poate arăta așa:

    linux vga=788 initrd=initrd.gz -- quiet

Poți adăuga [parametrii suplimentari][boot] pentru instalare:

    linux vga=788 initrd=initrd.gz netcfg/disable_dhcp=true
      modules=network-console -- quiet panic=1200

[boot]: https://www.debian.org/releases/stable/amd64/ch05s03.html.en

### Verificare proprietăți ale sistemului instalat

Arhitectura sistemului de operare (32 sau 64 biți):

    HOSTu:~# dpkg --print-architecture
    amd64  -- 64 biți (Intel/AMD)
    i386   -- 32 biți (Intel/AMD)
    sparc  -- 32 biți (Sun SPARC)

Setul implicit de codificare al caracterelor:

    HOSTu:~# grep -v '^\s*\(#.*\)\?$' /etc/locale.gen
    en_US.UTF-8 UTF-8

Nu utiliza un set diferit de **UTF-8** (ex. `en_US ISO-8859-1`) decât dacă știi ceea ce faci.

Lista pachetelor importante ce nu sunt instalate:

    HOSTu:~# aptitude search '~prequired!~i' '~pimportant!~i'
    p  dhcp3-client      - ISC DHCP server (transitional package)
    p  dhcp3-common      - ISC DHCP common files (transitional package)

Pachetele tranziționale pot fi ignorate fără probleme.


Configurare host
----------------

### Consolă «root» — editează ~/.bashrc

    # ~/.bashrc: executed by bash(1) for non-login shells.

    # Note: PS1 and umask are already set in /etc/profile. You should not
    # need this unless you want different defaults for root.
    # PS1='${debian_chroot:+($debian_chroot)}\h:\w\$ '
    # umask 022

    # You may uncomment the following lines if you want `ls' to be colorized:
    export LS_OPTIONS='--color=auto'
    eval "`dircolors`"
    alias ls='ls $LS_OPTIONS'
    alias ll='ls $LS_OPTIONS -l'
    alias l='ls $LS_OPTIONS -lA'

    # Some more alias to avoid making mistakes:
    alias rm='rm -i'
    alias cp='cp -i'
    alias mv='mv -i'

    alias apt='apt-get --purge'
    alias upd='apt-get update'
    alias upg='apt-get --purge dist-upgrade'
    alias rmdir='rmdir --ignore-fail-on-non-empty'

    HISTFILESIZE=200000
    HISTSIZE=60000

### Configurare «APT» — administratorul de pachete

Editează lista repozitoriilor de pachete din `/etc/apt/sources.list` pentru versiunea curentă stabilă:

    # [Debian GNU/Linux 6.0.6 _Squeeze_ 2012/11/20]
    # deb http://ftp.ro.debian.org/debian squeeze main

    # 6.0 main archives
    deb http://ftp.ro.debian.org/debian  squeeze  main contrib non-free
    deb http://ftp.de.debian.org/debian  squeeze  main
    deb http://security.debian.org  squeeze/updates  main contrib non-free

    # stable updates, previously known as 'volatile'
    deb http://ftp.ro.debian.org/debian  squeeze-updates  main contrib non-free
    deb http://ftp.de.debian.org/debian  squeeze-updates  main

    # backports from sid/testing
    deb http://ftp.de.debian.org/debian-backports  squeeze-backports  main contrib non-free
    deb http://backports.debian.org/debian-backports  squeeze-backports  main contrib non-free

Setează prioritatea pachetelor din principalele repozitorii suplimentare:

    cat > /etc/apt/preferences.d/backports <<__EOF__
    Package: *
    Pin: release o=Debian Backports,a=squeeze-backports
    Pin-Priority: 400
    __EOF__

    cat > /etc/apt/preferences.d/testing <<__EOF__
    Package: *
    Pin: release o=Debian,a=testing
    Pin-Priority: 300
    __EOF__

    cat > /etc/apt/preferences.d/unstable <<__EOF__
    Package: *
    Pin: release o=Debian,a=unstable
    Pin-Priority: 200
    __EOF__

Setează evenimente periodice de întreținere a administratorului de pachete **APT**:

    cat > /etc/apt/apt.conf.d/10periodic <<__EOF__
    APT::Periodic::AutocleanInterval "60";
    APT::Periodic::Download-Upgradeable-Packages "1";
    APT::Periodic::Update-Package-Lists "1";
    __EOF__

Acestea semnifică actualizarea automată a listei pachetelor din repozitoriile Debian, ștergerea pachetelor mai vechi de 60 de zile și descărcarea pachetelor noi (fără instalare).

Verifică dacă mai sunt disponibile alte actualizări de pachete:

    HOSTu:~# apt update && apt dist-upgrade

Notă: **apt** este un alias pentru `apt-get --purge` dar poți utiliza `aptitude` la fel de bine.

### Configurare «GRUB2» — încărcătorul sistemului de operare la „boot”

Editează fișierul `/etc/default/grub` pentru adăugare parametrii:

    GRUB_CMDLINE_LINUX_DEFAULT="quiet"
    GRUB_CMDLINE_LINUX="panic=1200 ipv6.disable=1"

    GRUB_GFXMODE=1024x768
    GRUB_GFXPAYLOAD_LINUX=keep

    GRUB_DISABLE_OS_PROBER="true"

Pentru sistemele „headless” ce sunt administrate la distanță prin **IPMI**, adaugă parametrii:

    GRUB_CMDLINE_LINUX_DEFAULT="quiet"
    GRUB_CMDLINE_LINUX="panic=1200 ipv6.disable=1 fb=false console=ttyS1,57600n8"
    [..]

    GRUB_TERMINAL=serial
    GRUB_SERIAL_COMMAND="serial --unit=1 --speed=57600 --word=8 --parity=no --stop=1"

### Configurare «mdadm» — serviciul „software RAID”

Este nevoie de o reconfigurare a serviciului `mdadm` pentru a activa doar partiția rădăcină din fișierul `initrd`:

    HOSTu:~# dpkg-reconfigure mdadm

                ┌────────────┤ Configuring mdadm ├────────────┐
                │ MD arrays needed for the root file system:  │
                │                                             │
                │ /dev/md/0__________________________________ │
                │                                             │
                │                   <Ok>                      │
                │                                             │
                └─────────────────────────────────────────────┘

Pentru restul întrebărilor nu trebuie să schimbi setările implicite.

### Instalare „kernel”-ul Linux optim

Pentru un sistem *686* instalează meta-pachetul „bigmem”:

    HOSTu:~# grep --color=auto pae /proc/cpuinfo && \
      apt install linux-image-686-bigmem

Instaleaza kernel-ul Linux din următoarea versiune stabilă:

    HOSTu:~# apt -t squeeze-backports install firmware-linux
    HOSTu:~# apt -t squeeze-backports install firmware-bnx2        # opțional

    HOSTu:~# sed -i "s|^\(GRUB_DEFAULT\)=.*$|\1=2|" /etc/default/grub

Comanda `sed` doar setează selecția implicită a kernel-ului din versiunea stabilă.
Alege meta-pachetul corespunzător 32 biți sau 64 biți:

    HOSTu:~# apt -t squeeze-backports install linux-image-amd64    # 64 biți
    The following extra packages will be installed:
      initramfs-tools linux-base linux-image-3.2.0-0.bpo.4-amd64

    HOSTu:~# apt -t squeeze-backports install linux-image-686-pae  # 32 biți

### Instalare microcod CPU

    HOSTu:~# apt install intel-microcode
    The following extra packages will be installed:
      microcode.ctl

    Download Intel CPU microcodes now?      <Yes>
    Applying Intel IA32 Microcode update... done.

Pentru procesoarele *AMD* instalează pachetul `amd64-microcode`.

#### Șterge legăturile simbolice din directorul rădăcină

    sed "s|^\(do_symlinks =\).*$|\1 no|" -i /etc/kernel-img.conf
    unlink /initrd.img
    unlink /initrd.img.old
    unlink /vmlinuz
    unlink /vmlinuz.old

### Configurare parametrii kernel via «sysctl»

    cat >> /etc/sysctl.d/local.conf <<__EOF__
    net.ipv4.conf.default.accept_redirects = 0
    net.ipv6.conf.default.accept_redirects = 0
    net.ipv4.conf.default.secure_redirects = 0
    net.ipv4.conf.default.send_redirects = 0
    net.ipv4.conf.default.accept_source_route = 0
    net.ipv6.conf.default.accept_source_route = 0

    net.ipv4.conf.all.accept_redirects = 0
    net.ipv6.conf.all.accept_redirects = 0
    net.ipv4.conf.all.secure_redirects = 0
    net.ipv4.conf.all.send_redirects = 0
    net.ipv4.conf.all.accept_source_route = 0
    net.ipv6.conf.all.accept_source_route = 0

    net.ipv4.conf.default.log_martians = 1
    net.ipv4.conf.all.log_martians = 1
    __EOF__
    
    cat >> /etc/sysctl.d/domainname.conf <<__EOF__
    kernel.domainname = $(dnsdomainname)
    __EOF__

Doar pentru poarta de rețea:

    cat >> /etc/sysctl.d/ip_forward.conf <<__EOF__
    net.ipv4.ip_forward = 1
    __EOF__


### Nume host

    D:~# sed "s|^$(hostname -s)$|$(hostname -f)|" -i /etc/hostname

#### Statistici pachete via „popcon”

Dezactivează depunerea de statistici „popcon” via email.

    echo "MAILTO=" >> /etc/popularity-contest.conf

##### -- #etc:default:rcS

Activează fixarea automată a problemelor detectate la verificarea sistemului de fișiere.

    sed "s|^#*\(FSCKFIX\)=.*$|\1=yes|" -i /etc/default/rcS

##### -- #etc:default:tmpfs

##### -- #etc:fstab

### Configurare serviciu „syslog”

Activează un jurnal cu toate mesajele de sistem, foarte util pentru depanare sistem la consolă (Alt+F12). Pentru sistemele „headless” (ex. SPARC) aceast lucru nu este posibil.

    cat >> /etc/rsyslog.d/tty12.conf <<__EOF__
    *.*                             /dev/tty12
    __EOF__

Setează un jurnal separat doar pentru mesajele trimise de serviciul „Cron”.

    CFGFILE=/etc/rsyslog.conf
    sed 's|\(authpriv.none\)[[:space:]]*\(-/var/log/syslog\)|\1;cron.none\t\2|' -i $CFGFILE
    sed 's|^#*\(cron.*cron.log\)|\1|' -i $CFGFILE


Configurare unelte
------------------

### Instalare și configurare «sendmail»

`sendmail` este doar o unealtă necesară pentru a trimite alerte de către unele servicii locale. Instalează `nullmailer` sau `ssmtp` sau orice alt pachet ce furnizează „sendmail” potrivit infrastructurii locale.

### Unelte de bază

    HOSTu:~# apt install deborphan debsums

    HOSTu:~# apt install bash-completion less parted mlocate vim vim-tiny- nano- zip unzip
    HOSTu:~# apt install atop htop iftop iotop lsof mtr-tiny tcpdump
    HOSTu:~# apt install ntpdate rsync telnet wget wput xauth

    HOSTu:~# apt install bridge-utils bzip2 dctrl-tools finger mc sudo \
      memtest86+			# nu la SPARC

#### Optimizare „VIM”

    cat > /etc/skel/.vimrc <<__EOF__
    :syntax on
    :set nofsync
    :set paste
    :set noautoindent
    __EOF__

    chmod -v a+r /etc/skel/.vimrc
    cp -va /etc/skel/.vimrc ~/.vimrc

### Unelte opționale

* „changetrack” — alertă modificări în sistem

* „rkhunter” — alertă bresă de securitate în sistem

* „smartmontools” — monitorizare disk

* „nut-client” — monitorizare **UPS**

* „ntpdate” — sincronizare dată și timp

* „tune2fs” — verificarea periodică a sistemului de fișiere

* „unattended-upgrades” — actualizare automată a pachetelor software

Configurarea inițială a sistemului (fară **X**) este completă!

    # update-grub && reboot
