PXE netboot
===========

„[Preboot eXecution Environment][PXE]” este o tehnologie de pornire a sistemelor de calcul folosind doar o interfață de rețea. Încărcarea sistemului de operare nu se face de pe unitatea de stocare locală a sistemului de calcul ci prin intermediul rețelei.

[PXE]: http://en.wikipedia.org/wiki/Preboot_Execution_Environment

PXE funcționează doar în rețeaua locală (LAN).
PXE se bazează pe serviciile **DHCP** și **TFTP**. Opțional în anumite configurații este utilizat serviciul **NFS**.


Instalare pachete software
--------------------------

Există cel puțin două metode de furnizare a serviciului PXE:

* sunt instalate și configurate pachetele `isc-dhcp-server tftpd-hpa`

* se instalează și configurează pachetul `dnsmasq` atât pentru DHCP cât și pentru TFTP

Serverul DHCP trebuie configurat să trimită parametrii suplimentari pentru „netboot”:

* adresa serverului TFTP  
* numele fișierului care trebuie încărcat de pe serverul TFTP

### Configurare serviciu DHCP (isc-dhcp-server)

Configurația suplimentară în serverul DHCP arată astfel:

    host NUME_STAȚIE {
      hardware ethernet hh:hh:hh:hh:hh:hh;
      fixed-address AAA.BBB.CCC.DDD;
      next-server T.F.T.P;
      filename "pxelinux.0";
    }

Alternativ, serverul DHCP poate fi configurat cu PXE pentru toate sistemele de calcul:

    subnet AAA.BBB.CCC.0 netmask 255.255.255.0 {
      [..]
      #range dynamic-bootp AAA.BBB.CCC.nnn AAA.BBB.CCC.mmm;
      if substring (option vendor-class-identifier, 0, 9) = "PXEClient" {
        next-server T.F.T.P;
        filename "/pxelinux.0";
      }
    }

### Configurare «dnsmasq»

    HOSTu:~# cat > /etc/dnsmasq.d/boot <<__EOF__
    dhcp-boot = pxelinux.0
    __EOF__

Implicit, adresa serverului **TFTP** este aceeași cu adresa serverului **DHCP**. Aceasta se poate schimba prin parametrii suplimentari.

    HOSTu:~# cat > /etc/dnsmasq.d/boot <<__EOF__
    dhcp-boot = pxelinux.0,tftp-servername,T.F.T.P
    __EOF__


Fișiere servite pe rețea prin TFTP
----------------------------------

Serverul TFTP este deja instalat, mai trebuie să adăugăm fișierele pentru PXE netboot:

    HOSTu:~# apt install syslinux-common      # ↦ pxelinux.0
    The following extra packages will be installed:
      libcrypt-passwdmd5-perl libdigest-sha1-perl

    HOSTu:~# cd /srv/tftp/
    HOSTu:/srv/tftp# cp -va /usr/lib/syslinux/pxelinux.0 .
    HOSTu:/srv/tftp# cp -va /usr/lib/syslinux/memdisk .
    HOSTu:/srv/tftp# mkdir -v pxelinux.cfg

    HOSTu:/srv/tftp# tar -zxvf netboot.tar.gz ./debian-installer/amd64/linux
    HOSTu:/srv/tftp# tar -zxvf netboot.tar.gz ./debian-installer/amd64/initrd.gz

Ultimul pas este crearea fișierului de configurare PXE specific unui sistem de calcul cu o adresă MAC a interfeței de rețea deja cunoscută:

    HOSTu:/srv/tftp# cat > pxelinux.cfg/01-hh-hh-hh-hh-hh-hh <<__EOF__
    DEFAULT install
    LABEL install
      KERNEL debian-installer/amd64/linux
      APPEND initrd=debian-installer/amd64/initrd.gz vga=788 -- quiet
    __EOF__

Dacă folosești DHCP static, poți folosi adresa IP în format hexazecimal pentru o configurare per-sistem sau per-subnet.
Alternativ, în `pxelinux.cfg/default` poți adăuga o configurație comună pentru toate sistemele de calcul.

Aceasta este structura de fișiere minimă pentru PXE netboot:

    /srv/tftp/
    ├── debian-installer
    │   └── amd64
    │       ├── initrd.gz
    │       └── linux
    ├── pxelinux.0
    └── pxelinux.cfg
        ├── 01-hh-hh-hh-hh-hh-hh (adresa MAC în format hexa — minuscule)
        ├── HHHHHHHH             (adresa IP sau subnet în format hexa)
        ├── HHHHHHH
        ├── ..
        ├── H
        └── default

### Parametrii pentru instalare semi-automată

În linia „append” se pot adăuga alți parametrii pentru instalare (până la 255 caractere).
De exemplu pentru inițializarea completă (peste o consolă serială pe COM2) a unei instalări la distanță prin SSH se pot adăuga parametrii următori (pe o singură linie):

    [..]
    SERIAL 1 57600n8

      [..]
      APPEND initrd=debian-installer/amd64/initrd.gz
        vga=normal fb=false console=ttyS1,57600n8
        country=$CO language=en locale=en_US console-keymaps-at/keymap=us
        hostname=$HOST domain=$DOMAIN
        netcfg/disable_dhcp=true
          netcfg/get_ipaddress=$IPADDR
          netcfg/get_netmask=$NETMASK
          netcfg/get_gateway=$GATEWAY
          netcfg/get_nameservers="$NAMESERVER1 $NAMESERVER2"
        mirror/country="$COUNTRY"
          mirror/http/hostname="ftp.${CO}.debian.org"
          mirror/http/directory="/debian"
          mirror/http/proxy=""
        modules=network-console
          network-console/password=r00t
          network-console/password-again=r00t
        cdrom-detect/eject=false -- quiet panic=1200

Astfel, instalarea poate fi continuată printr-o conexiune securizată SSH.

### Adăugare fișiere firmware în «initrd.gz»

    HOSTu:~# TMPDIR=$(mktemp -d)
    HOSTu:~# cd $TMPDIR

    HOSTu: tmp.XXXXXX # zcat /srv/tftp/debian-installer/amd64/initrd.gz | cpio -i
    HOSTu: tmp.XXXXXX # aptitude download firmware-bnx2
    HOSTu: tmp.XXXXXX # mkdir -v lib/firmware
    HOSTu: tmp.XXXXXX # dpkg-deb -x firmware-bnx2_0.28+squeeze1_all.deb bnx2
    HOSTu: tmp.XXXXXX # mv -v bnx2/lib/firmware/bnx2* lib/firmware/
    HOSTu: tmp.XXXXXX # rm -fr bnx2 firmware-bnx2_0.28+squeeze1_all.deb
    HOSTu: tmp.XXXXXX # find . -print0 | cpio -0 -H newc -o | gzip -c > ~/initrd.gz

    HOSTu:/srv/tftp# rm -fr $TMPDIR
    HOSTu:/srv/tftp# mv -vf ~/initrd.gz debian-installer/amd64/

### Încărcare „live CD” prin TFTP

Nu toate imaginile „live CD” pot fi încărcate cu succes de pe un server TFTP.
Se poate include imaginea ISO în fișierul `initrd.gz` folosind un script cum ar fi `livecd-iso-to-pxeboot`. În acest caz este necesară adăugarea parametrilor de pornire:

    LABEL [..]
      APPEND [..] root=/fisier_LiveCD.iso rootfstype=iso9660 rootflags=loop

Imaginile **ISOHYBRID** pot fi încărcate prin TFTP în majoritatea situațiilor folosind o configurație simplă:

    DEFAULT pxeboot
    LABEL pxeboot
      KERNEL memdisk
      APPEND iso initrd=/debian-6.0.5-amd64-i386-netinst.iso [raw]  [pause]

Principalele dezavantaje ale acestei metode sunt:

* timp mare de așteptare până când imaginea ISO este încărcată în memoria RAM

* nu se pot trimite automat parametrii de pornire

### Încărcare „live CD” prin NFS

    PROMPT 1
    DEFAULT install
    [..]

    LABEL grml64ful
      KERNEL grml/boot/grml64full/vmlinuz
      INITRD grml/boot/grml64full/initrd.img
      APPEND root=/dev/nfs rootfstype=nfs
        nfsroot=T.F.T.P:/srv/tftp/grml boot=live
        live-media-path=/live/grml64-full
        -- quiet  lang=us apm=power-off nomce noprompt noeject

    LABEL Ubuntu 11.04
      KERNEL ubuntu/11.04/casper/vmlinuz
      INITRD ubuntu/11.04/casper/initrd.lz
      APPEND netboot=nfs root=/dev/nfs
        nfsroot=T.F.T.P:/srv/tftp/ubuntu/11.04 boot=casper -- quiet

    ONERROR LOCALBOOT 0
