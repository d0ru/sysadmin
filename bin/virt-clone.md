virtinst :: 0.600.1-3 (admin)
=============================

[Virt Clone][acasă] — o unealtă pentru clonarea unui host virtual (eng. „guest”), existent dar inactiv.

[acasă]: http://virt-manager.org/

Pentru o clonare cât mai simplă este recomandată adăugarea spațiului implicit de stocare „default” la serviciul `libvirtd`.


Crează o clonă a unui host virtual existent
-------------------------------------------

Înainte de execuție hostul virtual ce urmează să fie clonat trebuie oprit.

    VOLD="nume-vechi"
    VNEW="nume-nou"
    virt-clone --file=/srv/virt/images/${VNEW}.img --original=$VOLD --name=$VNEW

#### O clonare rapidă fără modificare disk virtual

    VOLD="nume-vechi"
    VNEW="nume-nou"
    virt-clone --preserve-data -f /srv/virt/images/${VNEW}.img -o $VOLD -n $VNEW

În acest caz este nevoie de copierea manuală a disk-ului virtual pentru mașina nou creată:

    cd /srv/virt/images/ && cp -va ${VOLD}.img ${VNEW}.img

Acest lucru este util când se dorește ca imaginea disk a noului sistem creat să fie identică cu originalul.

### Adjustează proprietățile noului host virtual

Execută comanda `virsh edit $VNEW` pentru a corecta unele proprietăți are noului host virtual, cum ar portul serverului **VNC**.

    <graphics type='vnc' port='590XY' autoport='no' listen='0.0.0.0'>
