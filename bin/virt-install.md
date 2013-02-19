virtinst :: 0.600.1-3 (admin)
=============================

[Virt Install][acasă] — o unealtă pentru instalarea unui host virtual (eng. „guest”).

[acasă]: http://virt-manager.org/

Pentru o instalare cât mai simplă este recomandată adăugarea spațiului implicit de stocare „default” la serviciul `libvirtd`.


Crează un host virtual (eng. „guest”)
-------------------------------------

#### Pregătește nume host virtual

    NET=pro
    VT=k
    OS=d7
    SN=00

    HOST="$NET-$OS$VT$SN"

Observații:

* NET: codul de rețea (ex. `pro`)
* OS: codul sistemului de operare (ex. `d7` pentru *Debian 7.0*, `r6` pentru *RHEL 6.x*)
* VT: tipul mașinii virtuale (ex. `k` pentru *KVM*, `q` pentru *QEMU*, `x` pentru *Xen*, `vb` pentru *VirtualBox*, `vm` pentru *VMware*, `vs` pentru *VServer*, `vp` pentru *VirtualPC*, `vz` pentru *OpenVZ*)
* SN: 01..99 trebuie să fie unic în aceeași rețea (vezi NET) pentru a nu apărea conflict la migrare.

### Parametrii pentru instalare *Debian Linux 7.0*

    ARCH=amd64
    DISTRO=debian
    SUITE="wheezy"
    OS_VARIANT="debian$SUITE"

    COUNTRY="ro"
    LOCATION="http://ftp.${COUNTRY}.debian.org/debian/dists/$SUITE/main/installer-$ARCH"
    
    X_ARGS="netcfg/disable_dhcp=true -- quiet"
    unset CDROM FLOPPY

### Parametrii pentru instalare *Red Hat Enterprise Linux 6*

    ARCH=x86_64
    DISTRO=rhel
    RELEASE=6.3
    OS_VARIANT="rhel6"

    LOCATION="http://ftp.${DOMAIN:-$(dnsdomainname)}/$DISTRO/$RELEASE/os/$ARCH"
    KSTCFG="ks=http://ftp.${DOMAIN:-$(dnsdomainname)}/$DISTRO/${HOST}.ks"
    NETCFG="ip=$IPADDR netmask=$NETMASK gateway=$GATEWAY dns=$NAMESERVER1"

    X_ARGS="linux $KSTCFG $NETCFG"
    unset CDROM FLOPPY

Observații:

* trebuie creat repozitoriul **RHEL** pe baza discului **DVD** de instalare
* opțional, poți utiliza un fișier „kickstart” pentru o instalare automatizată
* opțional, poți specifica parametrii de rețea pentru o instalare automatizată

#### Parametrii opționali pentru instalare

    MEMORY=8192      # RAM size

    DISKSIZE=60      # virtual disk image size (RAW/QCOW2)
    SPARSE=false     # fully allocate the space for the virtual disk

    VIRTNET="-w bridge=br0"

    OS_ARCH="--arch=i686"

    FORCE="--force"

### Pornește instalare hostului virtual

    virt-install --name=$HOST $OS_ARCH --${VIRT_TYPE:-accelerate} $VIRTNET \
      --vcpus=${VCPUS:-2} --check-cpu --cpuset=auto --ram=${MEMORY:-2048} \
      --os-type=${OS_TYPE:-linux} --os-variant=${OS_VARIANT:-virtio26} \
      --disk pool=default,size=${DISKSIZE:-20},sparse=${SPARSE:-true} \
      --noautoconsole --vnc --vnclisten=0.0.0.0 --vncport=590${SN:-01} \
      --location=$LOCATION -x "$X_ARGS" $CDROM $FLOPPY $FORCE

Instalarea se poate finaliza prin conectarea la serviciul **VNC** de la portul `590SN`.
