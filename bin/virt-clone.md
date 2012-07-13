virtinst :: 0.600.1-2 (admin)
=============================

[virt-clone][home] este o unealtă pentru clonarea unui host virtual (numit și „guest”), existent dar inactiv.

[home]: http://virt-manager.org/


Pentru o clonare cât mai simplă este recomandată adăugarea spațiului implicit de stocare „default” la serviciul «libvirtd».


Crează o clonă a unui host virtual existent
-------------------------------------------

Înainte de execuție hostul virtual ce urmează să fie clonat trebuie oprit.

    VOLD="nume-vm"
    VNEW="${VOLD}CLONE"
    virt-clone --file=/srv/virt/images/${VNEW}.img --original=${VOLD} --name=${VNEW}

### O clonare rapidă fără modificare disk cu „SO” original

    VOLD="nume-vm"
    VNEW="${VOLD}CLONE"
    virt-clone --preserve-data -f /srv/virt/images/${VNEW}.img -o ${VOLD} -n ${VNEW}

Este nevoie de copierea manuală a disk-ului virtual pentru mașina nou creată:

    cd /srv/virt/images/ && cp -va ${VOLD}.img ${VNEW}.img

Acest lucru este util când se dorește ca sistemul de operare original să nu fie modificat în nici un fel în timpul clonării.
