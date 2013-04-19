roundcube :: 0.7.2-9 (web)
==========================

[Roundcube][acasă] — este o soluție webmail pentru serverul **IMAP**. Furnizează toate funcționalitățile așteptate de la un client email, inclusiv suport **MIME**, carte de adrese, administrare directoare și filtre mesaje.

[acasă]: http://www.roundcube.net


Instalare pachete
-----------------

Cerință pre-instalare:

* server web: `apache2`
* server **IMAP**: `dovecot-imapd`
* server **SMTP**: `postfix`
* server baze de date: `postgresql` | `mysql`.

Instalează toate pachetele disponibile:

    HOSTu:~# apt install roundcube roundcube-pgsql roundcube-plugins roundcube-plugins-extra
    The following extra packages will be installed:
      aspell aspell-en dbconfig-common fontconfig-config javascript-common
      libaspell15 libfontconfig1 libgd2-xpm libicu48 libjpeg8 libjs-jquery
      libjs-jquery-mousewheel libjs-jquery-ui libmcrypt4 libpng12-0 libxpm4
      php-auth php-auth-sasl php-mail-mime php-mail-mimedecode php-mdb2
      php-mdb2-driver-pgsql php-net-sieve php-net-smtp php-net-socket php-pear
      php5-gd php5-intl php5-mcrypt php5-pgsql php5-pspell roundcube-core tinymce
      ttf-dejavu-core wwwconfig-common
    
    Configure database for roundcube with dbconfig-common?  Yes
    Database type to be used by roundcube:                  pgsql

    PostgreSQL application password for roundcube:          <blank>

Notă: **apt** este un alias pentru `apt-get --purge` dar poți utiliza `aptitude` la fel de bine.

Module suplimentare:

* securitate: `fail2ban`


Configurare serviciu
--------------------

### Setează opțiuni prin reconfigurare pachet

    HOSTu:~# dpkg-reconfigure roundcube-core
      IMAP server(s) used with RoundCube:            localhost

      Default language:                              en_US | ro_RO
      Reinstall database for roundcube?              No
      Web server(s) to configure automatically:      (none)
      Should the webserver(s) be restarted now?      No

    HOSTu:~# ls -l /etc/apache2/conf.d/
    [..]
    lrwxrwxrwx [..] javascript-common.conf -> /etc/javascript-common/javascript-common.conf
    lrwxrwxrwx [..] roundcube -> /etc/roundcube/apache.conf
    
    HOSTu:~# unlink /etc/apache2/conf.d/javascript-common.conf
    HOSTu:~# unlink /etc/apache2/conf.d/roundcube

Am șters aceste legături pentru a dezactiva [configurația implicită][618699].

[618699]: http://bugs.debian.org/618699

### Setează limita maximă a dimensiunii fișierelor atașate

Editează fișierul `/etc/roundcube/htaccess` să conțină:

    <IfModule mod_php5.c>
      [..]
      php_value       upload_max_filesize     20M

      [..]
      # http://bugs.php.net/bug.php?id=30766
      #php_value      mbstring.func_overload  0
    </IfModule>

Pentru setări suplimentare editează `/etc/roundcube/main.inc.php` să conțină:

    // LOGGING/DEBUGGING
    $rcmail_config['log_logins'] = true;
    $rcmail_config['log_session'] = true;

    // IMAP
    $rcmail_config['default_host'] = 'localhost';
    // SMTP
    $rcmail_config['smtp_server'] = 'localhost';

    // SYSTEM
    $rcmail_config['force_https'] = true;
    $rcmail_config['login_autocomplete'] = 1;
    $rcmail_config['session_domain'] = '<DOMAIN.tld>';
    //$rcmail_config['ip_check'] = true;
    $rcmail_config['mail_domain'] = '%d';
    $rcmail_config['password_charset'] = 'UTF-8';
    $rcmail_config['sendmail_delay'] = 20;
    $rcmail_config['http_received_header'] = true;
    $rcmail_config['send_format_flowed'] = false;
    $rcmail_config['dont_override'] = array('timezone', 'dst_active', 'mdn_requests',
                                        'mime_param_folding', 'force_7bit', 'mdn_default',
                                        'dsn_default', 'strip_existing_sig',
                                        'default_charset', 'logout_expunge');
    $rcmail_config['identities_level'] = 1;

    // PLUGINS
    $rcmail_config\['plugins'\] = array('archive', 'dkimstatus', 'emoticons', 'fail2ban',
                                    'markasjunk', 'new_user_dialog', 'vcard_attachments');

    // USER INTERFACE
    $rcmail_config['list_cols'] = array('flag', 'status', 'attachment', 'subject',
                                        'from', 'date', 'size');
    $rcmail_config['max_pagesize'] = 6000;

    // USER PREFERENCES
    $rcmail_config['default_charset'] = 'UTF-8';
    $rcmail_config['pagesize'] = 600;
    $rcmail_config['show_images'] = 1;
    $rcmail_config['htmleditor'] = 2;
    $rcmail_config['preview_pane'] = true;
    $rcmail_config['logout_expunge'] = true;
    $rcmail_config['mime_param_folding'] = 0;
    $rcmail_config['skip_deleted'] = true;
    $rcmail_config['check_all_folders'] = true;
    $rcmail_config['autoexpand_threads'] = 2;
    $rcmail_config['mdn_requests'] = 2;

Aceste pluginuri activate nu au fișier de configurare sau configurația implicită este destul de bună.

### Activează modulul «show_additional_headers»

    PLUGIN="show_additional_headers"
    grep --color -w "$PLUGIN" $MAINCFG || \
      sed "s|\(rcmail_config\['plugins'\] = array(.*\));$|\1, '$PLUGIN');|" -i $MAINCFG

Adaugă următoarea configurație în `/etc/roundcube/main.inc.php`:
   
    # plugin: show_additional_headers
    $rcmail_config['show_additional_headers'] = array('Reply-To', 'Followup-To', 'List-Id',
                                                  'Sender', 'X-Sender', 'Delivered-To',
                                                  'User-Agent', 'X-Mailer', 'Organization');

### Activează modulul «managesieve»

    PLUGIN="managesieve"
    grep --color -w "$PLUGIN" $MAINCFG || \
      sed "s|\(rcmail_config\['plugins'\] = array(.*\));$|\1, '$PLUGIN');|" -i $MAINCFG

Adaugă următoarea configurație în `/etc/roundcube/plugins/managesieve/config.inc.php`:

    <?php

    // managesieve server port
    $rcmail_config['managesieve_port'] = 4190;

    // managesieve server address, default is localhost
    $rcmail_config['managesieve_host'] = 'localhost';

    // authentication method
    $rcmail_config['managesieve_auth_type'] = 'PLAIN';

    // use or not TLS for managesieve server connection
    $rcmail_config['managesieve_usetls'] = false;

    // the name of the script which will be used when there's no user script
    $rcmail_config['managesieve_script_name'] = 'roundcube';

    // Sieve RFC says that we should use UTF-8 endcoding for mailbox names
    $rcmail_config['managesieve_mbox_encoding'] = 'UTF-8';

    ?>

Acest fișier este adaptat după `/usr/share/roundcube/plugins/managesieve/config.inc.php.dist`.

### Activează modulul «squirrelmail_usercopy»

Acest modul este util dacă migrezi de la `squirrelmail` la `roundcube` — importă automat unele preferințe ale utilizatorilor și cartea de adrese în baza de date *Roundcube* la prima conectare.

    PLUGIN="squirrelmail_usercopy"
    grep --color -w "$PLUGIN" $MAINCFG || \
      sed "s|\(rcmail_config\['plugins'\] = array(.*\));$|\1, '$PLUGIN');|" -i $MAINCFG

    chmod ug+r /var/lib/squirrelmail/data

Adaugă următoarea configurație în `/etc/roundcube/plugins/squirrelmail_usercopy/config.inc.php`:

    // Driver - 'file' or 'sql'
    $rcmail_config['squirrelmail_driver'] = 'file';
    
    // full path to the squirrelmail data directory
    $rcmail_config['squirrelmail_data_dir'] = '/var/lib/squirrelmail/data';
    $rcmail_config['squirrelmail_data_dir_hash_level'] = 0;
    
    // identities_level option value for squirrelmail plugin
    // With this you can bypass/change identities_level checks
    // for operations inside this plugin. See #1486773
    $rcmail_config['squirrelmail_identities_level'] = null;
    
    // set to false if your squirrelmail config setting $edit_identity has been true
    $rcmail_config['squirrelmail_set_alias'] = false;

### Adaugă un portal web dedicat pentru *webmail*

    <VirtualHost *:80>
    	ServerAdmin	webmaster@DOMAIN.tld
    	ServerName	mail.DOMAIN.tld
    	ServerAlias	mail
    	RedirectMatch ^/(.*)	https://mail.DOMAIN.tld/$1
    </VirtualHost>

    <VirtualHost mail.DOMAIN.tld:443>
    	ServerAdmin	webmaster@DOMAIN.tld
    	ServerName	mail.DOMAIN.tld
    	RedirectMatch ^/$	https://mail.DOMAIN.tld/roundcube/
    	RedirectMatch ^/roundcube/..*/$	https://mail.DOMAIN.tld/roundcube/

    	SSLEngine		On
    	SSLCertificateFile	/etc/ssl/certs/mail.DOMAIN.tld.crt
    	SSLCertificateKeyFile	/etc/ssl/private/mail.DOMAIN.tld.key
    	SSLCACertificateFile	/usr/share/ca-certificates/cacert.org/cacert.org.crt

    	DocumentRoot /var/www/mail.DOMAIN.tld/
    	<Directory "/var/www/mail.DOMAIN.tld/">
    		AllowOverride None
    		Options FollowSymLinks
    		Order allow,deny
    		Allow from all
    	</Directory>

    	Alias /javascript/ /usr/share/javascript/
    	<Directory "/usr/share/javascript/">
    		Options FollowSymLinks MultiViews
    	</Directory>

    	Alias /roundcube/program/js/tiny_mce/ /usr/share/tinymce/www/
    	<Directory "/usr/share/tinymce/www/">
    		AllowOverride None
    		Options FollowSymLinks MultiViews
    		Order allow,deny
    		Allow from all
    	</Directory>

    	Alias /roundcube/ /var/lib/roundcube/
    	<Directory "/var/lib/roundcube/">
    		Options +FollowSymLinks
    		# This is needed to parse /var/lib/roundcube/.htaccess. See its
    		# content before setting AllowOverride to None.
    		AllowOverride All
    		Order allow,deny
    		Allow from all
    	</Directory>

    	# Protecting basic directories:
    	<Directory "/var/lib/roundcube/config/">
    		Options -FollowSymLinks
    		AllowOverride None
    	</Directory>
    	<Directory "/var/lib/roundcube/logs/">
    		Options -FollowSymLinks
    		AllowOverride None
    		Order allow,deny
    		Deny from all
    	</Directory>
    	<Directory "/var/lib/roundcube/temp/">
    		Options -FollowSymLinks
    		AllowOverride None
    		Order allow,deny
    		Deny from all
    	</Directory>

    	BrowserMatch "MSIE [2-6]" \
    		nokeepalive ssl-unclean-shutdown \
    		downgrade-1.0 force-response-1.0
    	# MSIE 7 and newer should be able to use keepalive
    	BrowserMatch "MSIE [17-9]" ssl-unclean-shutdown

    	ErrorLog ${APACHE_LOG_DIR}/error.log

    	# Possible values include: debug, info, notice, warn, error, crit,
    	# alert, emerg.
    	LogLevel warn

    	CustomLog ${APACHE_LOG_DIR}/access_mail-DOMAIN-tld.log combined
    	ServerSignature On
    </VirtualHost>

Activează portalul *webmail* și repornește serviciul *apache2*:

    HOSTu:~# a2enmod ssl
    HOSTu:~# a2ensite mail.DOMAIN.tld
    HOSTu:~# service apache2 restart


Fapte cunoscute
---------------

### W: empty value in steps/mail/compose.inc on line 239

Mesajul complet din `/var/log/roundcube/errors` este:

     PHP Warning:  Creating default object from empty value in
       /usr/share/roundcube/program/steps/mail/compose.inc on line 239

Această [problemă][1488404] apare de fiecare dată când este trimis un mesaj. *Roundcube* v0.8 conține un [fix][252d2745].

[1488404]: http://trac.roundcube.net/ticket/1488404
[252d2745]: http://trac.roundcube.net/changeset/252d2745/github#file0
