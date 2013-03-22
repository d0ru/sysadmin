ldap-account-manager 3.7-2 (web)
================================

[LDAP Account Manager][acasă] — o aplicație web pentru administrarea unui server **LDAP**.

[acasă]: https://www.ldap-account-manager.org/

Scurt glosar:

* **LDAP** — „Lightweight Directory Access Protocol”, [*RFC 4510*][rfc4510]

[rfc4510]: http://tools.ietf.org/html/rfc4510


Instalare pachete
-----------------

Cerință pre-instalare:

* server web: `apache2`
* server **LDAP**: `slapd`

Instalează toate pachetele necesare:

    HOSTu:~# apt install ldap-account-manager
    The following extra packages will be installed:
      php-fpdf php5-gd php5-ldap

Notă: **apt** este un alias pentru `apt-get --purge` dar poți utiliza `aptitude` la fel de bine.


Configurare serviciu
--------------------

Interfața web oferită de **LAM** este disponibilă implicit la adresa `http://host.domain.tld/lam/`, unde *host.domain.tld* este adresa hostului pe care a host instalat serviciul.
Serviciul **OpenLDAP** este de obicei instalat pe același host, însă nu este obligatoriu.

### Editare profil server LDAP

Profilul implicit este accesat din pagina web principală prin legăturile *LAM configuration* ↦ *Edit server profiles*. Din pagina *Manage server profiles* poți adăuga un profil nou sau utiliza profilul implicit `lam` (parola *lam*).

Detaliile de conectare la serverului **LDAP** se scriu în tabul *General settings* astfel:

    Server settings
    ---------------
      + Tree suffix: dc=DOMAIN,dc=TLD
      + Activate TLS: No

    Security settings
    -----------------
      + List of valid users: cn=admin,dc=DOMAIN,dc=TLD

În tabul *Account types* adjustează toate intrările referitoare la *LDAP suffix* pentru domeniul tău.
