lvm2 :: 2.02.95-4 (admin)
=========================

[LVM2][home] este un set de unelte pentru controlul dispozitivelor LVM.

[home]: http://sources.redhat.com/lvm2/


Controlul grupurilor LVM
------------------------

#### Inițializarea volumelor fizice pentru LVM

    pvcreate /dev/sda4
    pvcreate /dev/sda11 /dev/sda12 /dev/sda13 /dev/sda14

Acest pas este opțional în crearea unui grup LVM.

#### Listarea tuturor partițiilor LVM

    pvs

#### Crearea unui grup LVM

    vgcreate <nume> <listă partiții alocate grupului LVM>
    vgcreate vgsrv /dev/sda7 /dev/sda8 /dev/sda9

Dacă volumele fizice nu au fost inițializate explicit cu `pvcreate`, vor fi inițializate automat de `vgcreate`.

#### Listarea tuturor grupurilor LVM

    vgs

#### Adăugarea sau extinderea unui grup LVM

    vgextend <nume> <listă partiții>
    vgextend vgsrv /dev/sda10

#### Redenumește un grup LVM

    vgrename <nume> <nume nou>


Controlul volumelor LVM
-----------------------

#### Crearea unui volum LVM

    lvcreate -L <dimensiune> -n <nume> <grup LVM>
    lvcreate -L 60G -n lvdb  vgsrv
    lvcreate -L 60G -n lvwww vgsrv

<grup LVM> trebuie să fie un grup LVM existent.

#### Listarea tuturor volumelor LVM

    lvs

#### Redimensionarea unui volum LVM

    lvresize -L <dimensiune> /dev/mapper/<grup LVM>-<volum LVM>
    lvresize -L 128G /dev/mapper/vgsrv-lvwww

#### Redenumește un volum LVM

    lvrename <grup LVM> <nume> <nume nou>
