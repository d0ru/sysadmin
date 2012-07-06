sed :: 4.2.1-10 (utils)
=======================

`sed` (eng. „stream editor”) este o unealtă foarte utilă în prelucrarea automată a unui text.

Oricare din aceste comenzi pot fi înlănțuite într-o singură execuție:

    sed "comanda1;comanda2;..;comandaN" ..

Dacă vrei să modifici fișierul original se adaugă parametrul `-i"[sufix]"`. Menționarea unui sufix face ca fișierul original să fie copiat într-un fișier nume+sufix („backup”).


Comenzi de modificare a textului
--------------------------------

#### Adaugă caracterul # la începutul liniilor ce încep cu un text dat

    sed "s%^TEXT%#&%" FILENAME
    sed "s%^#*\(TEXT\)%#\1%" FILENAME

Am utilizat caracterul separator `%` deoarece acesta apare foarte rar în textul procesat sau adăugat.

#### Adaugă alt parametru la o listă existentă

    sed "s%TEXT(\(.*\));$%TEXT(\1, 'ALTPARAM');%" FILENAME


Comenzi de extragere valori din text
------------------------------------

#### Extrage o valoare folosind expresii regulate

    sed -n 's%NUMEVAR="\(.*\)"%\1%p' FILENAME
    sed -n "s%^\s*$KEYWORD\s\+\(.*\)%\1%p" $CONF


Comenzi de ștergere selectivă a textului
----------------------------------------

#### Șterge un bloc de linii după număr

    sed '6,20d' FILENAME
    sed '6,$d' FILENAME
    sed '$d' FILENAME         # șterge ultima linie

#### Șterge toate liniile goale

    sed '/^$/d' FILENAME
    sed '/./!d' FILENAME

#### Șterge toate liniile ce conțin un șablon

    sed '/REGEXP/d' FILENAME
    sed "/^--.*--$/d" FILENAME

#### Șterge toate aparițiile unui paragraf dintr-un fișier

    sed "/TEXT-ÎNCEPUT/,/TEXT-SFÂRȘIT/d" FILENAME
    sed "/^--/,/--$/d" FILENAME
