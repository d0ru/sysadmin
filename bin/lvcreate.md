lvm2 :: 2.02.66-4 (admin)
=========================

`lvcreate` este o unealtă pentru crearea unui volum LVM într-un grup LVM existent.


Crearea unui volum LVM
----------------------

    lvcreate -L <DIMENSIUNE> -n <NUME> <GRUP-LVM>
    lvcreate -L 60G -n lvdb  vgsrv
    lvcreate -L 60G -n lvwww vgsrv
