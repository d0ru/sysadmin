nagios-nrpe-server :: 2.12-4 (net)
==================================

NRPE (eng. „Nagios Remote Plugin Executor”) permite execuția automată a unor comenzi de pe un sistem central pe sistemele monitorizate.


Instalare pachete
-----------------

    HOSTu:~# apt install nagios-plugins
    The following extra packages will be installed:
      dnsutils fping libldap-2.4-2 libmysqlclient16 libnet-snmp-perl libpq5
      libradiusclient-ng2 libsasl2-2 libsasl2-modules libtalloc2 libwbclient0
      mysql-common nagios-plugins-basic nagios-plugins-standard qstat samba-common
      samba-common-bin smbclient

    HOSTu:~# apt install nagios-nrpe-server

Notă: **apt** este un alias pentru `apt-get --purge` dar poți utiliza `aptitude` la fel de bine.


Configurare serviciu
--------------------

#### Permisiuni, activare comenzi prin «NRPE»

    NAGIOSSERVER="adresa-IP-a-serverului-Nagios"
    sed "s|^#*\(allowed_hosts\)=.*$|\1=127.0.0.1,${NAGIOSSERVER}|" -i /etc/nagios/nrpe.cfg$SUFFIX
    sed "s|^#*\(dont_blame_nrpe\)=.*$|\1=1|" -i /etc/nagios/nrpe.cfg$SUFFIX

Variabila „SUFFIX” este utilă doar la instalarea pachetului cu un fișier de configurare diferit:

    SUFFIX=".dpkg-dist"

#### Comenzi utilizate pentru monitorizare

    cat > /etc/nagios/nrpe.d/common.cfg <<__EOF__
    command[disk]=/usr/lib/nagios/plugins/check_disk -w \$ARG1$ -c \$ARG2$ -W \$ARG1$ -K \$ARG2$ -e -L
    command[disk-p]=/usr/lib/nagios/plugins/check_disk -w \$ARG1$ -c \$ARG2$ -W \$ARG1$ -K \$ARG2$ -E -p \$ARG3$
    command[disk-x]=/usr/lib/nagios/plugins/check_disk -w \$ARG1$ -c \$ARG2$ -W \$ARG1$ -K \$ARG2$ -e -L -x \$ARG3$
    command[disk-X]=/usr/lib/nagios/plugins/check_disk -w \$ARG1$ -c \$ARG2$ -W \$ARG1$ -K \$ARG2$ -e -L -X \$ARG3$
    command[disk-X2]=/usr/lib/nagios/plugins/check_disk -w \$ARG1$ -c \$ARG2$ -W \$ARG1$ -K \$ARG2$ -e -L -X \$ARG3$ -X \$ARG4$
    command[disk-Xx]=/usr/lib/nagios/plugins/check_disk -w \$ARG1$ -c \$ARG2$ -W \$ARG1$ -K \$ARG2$ -e -L -X \$ARG3$ -x \$ARG4$
    command[load]=/usr/lib/nagios/plugins/check_load -w \$ARG1$ -c \$ARG2$
    command[load-r]=/usr/lib/nagios/plugins/check_load -w \$ARG1$ -c \$ARG2$ -r
    command[ntp_time]=/usr/lib/nagios/plugins/check_ntp_time -w \$ARG1$ -c \$ARG2$ -q -H \$ARG3$
    command[mailq]=/usr/lib/nagios/plugins/check_mailq -w \$ARG1$ -c \$ARG2$ -M \$ARG3$
    command[mysql]=/usr/lib/nagios/plugins/check_mysql -u \$ARG1$ -p \$ARG2$
    command[pgsql]=/usr/lib/nagios/plugins/check_pgsql -w \$ARG1$ -c \$ARG2$
    command[pgsql-d]=/usr/lib/nagios/plugins/check_pgsql -w \$ARG1$ -c \$ARG2$ -d \$ARG3$
    command[procs]=/usr/lib/nagios/plugins/check_procs -w \$ARG1$ -c \$ARG2$
    command[procs-a]=/usr/lib/nagios/plugins/check_procs -c \$ARG1$ -a \$ARG2$
    command[procs-C]=/usr/lib/nagios/plugins/check_procs -c \$ARG1$ -C \$ARG2$
    command[procs-P]=/usr/lib/nagios/plugins/check_procs -c \$ARG1$ -P \$ARG2$
    command[procs-s]=/usr/lib/nagios/plugins/check_procs -w \$ARG1$ -c \$ARG2$ -s \$ARG3$
    command[tcp]=/usr/lib/nagios/plugins/check_tcp -w \$ARG1$ -c \$ARG2$ -H \$ARG3$ -p \$ARG4$
    command[users]=/usr/lib/nagios/plugins/check_users -w \$ARG1$ -c \$ARG2$
    __EOF__

#### Start serviciu

    HOSTu:~# service nagios-nrpe-server restart

#### Verificare și testare

    /usr/lib/nagios/plugins/check_disk -w 6% -c 2% -W 6% -K 2% -e -X tmpfs

    /usr/lib/nagios/plugins/check_nrpe -H $HOST -c disk-X -a 6% 2% tmpfs
    /usr/lib/nagios/plugins/check_nrpe -H $HOST -c disk-p -a 6% 2% /
