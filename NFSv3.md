nfs-kernel-server 1:1.2.2-4squeeze2 (net)
=========================================

Serviciul **NFS** funcționează în „kernel-space” — este oferit direct de către sistemul de operare Linux și nu de către o aplicație server din „user-space”.

Notă: este recomandată sincronizarea regulată a timpului pe TOATE nodurile din rețea. Fără o sincronizarea a ceasului între client-server, **NFS** poate introduce întârzieri nedorite!


Instalare pachete server NFS
----------------------------

    HOSTu:~# apt install nfs-kernel-server
    The following extra packages will be installed:
      libevent-1.4-2 libgssglue1 libldap-2.4-2 libnfsidmap2 librpcsecgss3
      libsasl2-2 libsasl2-modules nfs-common portmap

Notă: **apt** este un alias pentru `apt-get --purge` dar poți utiliza `aptitude` la fel de bine.


Instalare pachete client NFS
----------------------------

Pentru un host Debian:

    HOSTu:~# apt install nfs-common
    The following extra packages will be installed:
      libevent-1.4-2 libgssglue1 libldap-2.4-2 libnfsidmap2 librpcsecgss3
      libsasl2-2 libsasl2-modules portmap

Pentru un host Fedora:

    [HOSTu ~]# yum install nfs-utils
    Installing for dependencies:
      libevent libgssapi nfs-utils-lib portmap


Configurare server NFSv3
------------------------

Adaugă în fișierul de configurare `/etc/exports` directorul pe care vrei să-l exporți prin **NFSv3**:

    /home  *(rw,sec=none:sys:krb5:krb5i:krb5p,no_subtree_check,no_root_squash)

    /srv/pub   *(ro,async,all_squash,insecure,no_subtree_check)
    /srv/tftp  *(ro,async,no_subtree_check,no_root_squash)
    /srv/tftp/HOST  HOST.DOMAIN(rw,sync,no_subtree_check,no_root_squash)

    /srv/nfs/backups/HOST  HOST.DOMAIN(rw,sync,no_subtree_check,no_root_squash)
    /srv/nfs/ftp  FTPHOST(rw,all_squash,anonuid=8000,anongid=8000)

Opțiunile implicite pentru exporturile **NFSv3** sunt:

    ro,sync,wdelay,hide,nocrossmnt,secure,root_squash,no_all_squash,
    no_subtree_check,secure_locks,acl,anonuid=65534,anongid=65534

Pentru activarea modificărilor efectuate în configurația serviciului **NFS** se poate utiliza următoarea comandă:

    HOSTu:~# exportfs -vr
    exporting *:/home
    exporting *:/srv/pub
    exporting *:/srv/tftp
    exporting [..]

Pentru dezactivarea temporară a serviciului **NFS** este suficientă ștergerea tuturor exporturilor:

    HOSTu:~# exportfs -vau


Configurare client NFSv3
------------------------

Pentru a testa montarea unui export **NFSv3** se poate utiliza comanda:

    HOSTu:~# mount.nfs NFSHOST:/home /mnt/nfs -v [-o <nfs_options>]

Pentru a salva permanent exporturile **NFSv3** editează fișierul de configurare `/etc/fstab` astfel:

    # <file system> <mount point>   <type>  <options>       <dump>  <pass>
    NFSHOST:/home   /home           nfs     relatime,nfsvers=3,proto=tcp,intr  0 2
    NFSHOST:/srv/nfs/backups/HOST  /srv/backups  nfs  relatime,nfsvers=3,proto=tcp,intr  0 2
