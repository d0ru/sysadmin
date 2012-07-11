tftpd-hpa :: 5.2-2 (net)
========================

„[Trivial File Transfer Protocol][TFTP]” este un protocol de transfer fișiere rapid folosit în principal pentru a servi imagini „boot” prin rețea către alte sisteme de calcul (tehnologia **PXE**).

[TFTP]: http://en.wikipedia.org/wiki/Trivial_File_Transfer_Protocol

Diferența esențială dintre **FTP** și **TFTP** este că acesta din urmă nu are nici un fel de autorizare la conectare.


Configurare serviciu TFTP
-------------------------

Implicit serviciul TFTP este configurat fără permisiunea de scriere. Aceasta se poate schimba prin adăugarea unui parametru la pornire.

    HOSTu:~# sed 's%\(TFTP_OPTIONS=".*\)"$%\1 --create"%' -i /etc/default/tftpd-hpa
    HOSTu:~# service tftpd-hpa restart
