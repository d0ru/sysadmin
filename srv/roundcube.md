roundcube :: 0.7.1-1~bpo60+1 (web)
==================================

[Roundcube][1] —  is a skinnable AJAX based webmail solution for IMAP servers.
It provides full functionality expected from an e-mail client, including MIME support, address book, folder manipulation and message filters.

[1]: http://www.roundcube.net


Install software packages
-------------------------

Server Requirements
The following packages needs to be installed and configured prior to `roundcube` installation:

* web server: `apache2` and `php-auth` (which depends on `php-pear php5-cli`)
* IMAP server: `dovecot-imapd`

Set packages PIN for selecting higher versions from backports during installation:

    HOSTu:~# SPKG=roundcube
    ARCHIVE=squeeze-backports
    ORIGIN="Debian Backports"
    
    : > /etc/apt/preferences.d/${SPKG}.pref
    for t in $(apt-cache dumpavail | grep-dctrl -S $SPKG -X -sPackage | cut -d: -f2) libjs-jquery; do
      grep -q "Package: $t$" /etc/apt/preferences.d/${SPKG}.pref && continue
      printf "Package: $t\nPin: release o=$ORIGIN,a=$ARCHIVE\nPin-Priority: 600\n\n"
      sync
    done > /etc/apt/preferences.d/${SPKG}.pref

Install a recommended set of **roundcube** related packages:

    HOSTu:~# apt install roundcube roundcube-plugins
    
    The following extra packages will be installed:
      dbconfig-common javascript-common libjs-jquery libjs-jquery-ui libmcrypt4
      libsqlite0 php-auth php-auth-sasl php-mail-mime php-mail-mimedecode php-mdb2
      php-mdb2-driver-sqlite php-net-smtp php-net-socket php-pear php5-intl
      php5-mcrypt php5-pspell php5-sqlite roundcube-core roundcube-sqlite sqlite
      tinymce wwwconfig-common
    
    Configure database for roundcube with dbconfig-common?  Yes     (default)
    Database type to be used by roundcube:                  sqlite  (default)

Note: **apt** is an alias for `apt-get` but you can use `aptitude` as well.

After install a short reconfiguration is necessary:

    HOSTu:~# dpkg-reconfigure roundcube-core
    
    Default language:                              en_US | ro_RO
    Reinstall database for roundcube?              No
    Web server(s) to configure automatically:      (none)
    Should the webserver(s) be restarted now?      No

This should disable the default configuration, see [Debian bug#618699][2].
If these symbolic links are present you should manually remove them.

    HOSTu:~# ls -l /etc/apache2/conf.d/
    [..]
    lrwxrwxrwx [..] javascript-common.conf -> /etc/javascript-common/javascript-common.conf
    lrwxrwxrwx [..] roundcube -> /etc/roundcube/apache.conf
    
    HOSTu:~# unlink /etc/apache2/conf.d/javascript-common.conf
    HOSTu:~# unlink /etc/apache2/conf.d/roundcube

[2]: http://bugs.debian.org/618699


Configure Roundcube service
---------------------------

Small changes to _htaccess_ global configuration file used by webmail virtual host. This will increase the maximum file size for attachments from *5MB* to *20MB*.

    SUFFIX=".dpkg-dist"                             # only on packages upgrade
    
    sed "s%\(upload_max_filesize\).*$%\1\t20M%" -i /etc/roundcube/htaccess$SUFFIX
    sed "s%^#*\(php_value\s*mbstring.func_overload\)%#\1%" -i /etc/roundcube/htaccess$SUFFIX
    sed "s%\(Header\) append \(Cache-Control\)%\1 merge \2%" -i /etc/roundcube/htaccess$SUFFIX
    
    [ -z "$SUFFIX" ] || mv -vf /etc/roundcube/htaccess$SUFFIX /etc/roundcube/htaccess

### Enable multiple Roundcube plugins

On upgrades it's practical to preserve the old configuration file and commit all changes to the new config — for this I use the `$SUFFIX` shell variable. For a fresh install **don't** set the SUFFIX variable.

    SUFFIX=".ucf-dist"                             # only on packages upgrade
    MAINCFG="/etc/roundcube/main.inc.php$SUFFIX"
    
    sed "s%\(rcmail_config\['plugins'\] =\) .*$%\1 array('archive', 'emoticons', 'markasjunk', 'new_user_dialog', 'vcard_attachments');%" -i $MAINCFG

These Roundcube plugins don't have a configuration file or the default configuration is good enough.

### Several adjustments to the default configuration

On upgrades it's practical to preserve the old configuration file and commit all changes to the new config — for this I use the `$SUFFIX` shell variable. For a fresh install **don't** set the SUFFIX variable.

    SUFFIX=".ucf-dist"                             # only on packages upgrade
    MAINCFG="/etc/roundcube/main.inc.php$SUFFIX"
    
    ## LOGGING/DEBUGGING
    sed "s%\(rcmail_config\['log_logins'\] =\) .*$%\1 true;%" -i $MAINCFG
    sed "s%\(rcmail_config\['log_session'\] =\) .*$%\1 true;%" -i $MAINCFG
    
    ## IMAP & SMTP
    sed "s%\(rcmail_config\['default_host'\] =\) .*$%\1 'localhost';%" -i $MAINCFG
    sed "s%\(rcmail_config\['smtp_server'\] =\) .*$%\1 'localhost';%" -i $MAINCFG
    
    ## SYSTEM
    sed "s%\(rcmail_config\['force_https'\] =\) .*$%\1 true;%" -i $MAINCFG
    sed "s%\(rcmail_config\['login_autocomplete'\] =\) .*$%\1 2;%" -i $MAINCFG
    sed "s%\(rcmail_config\['session_domain'\] =\) .*$%\1 '$(dnsdomainname)';%" -i $MAINCFG
    sed "s%\(rcmail_config\['mail_domain'\] =\) .*$%\1 '\%d';%" -i $MAINCFG
    sed "s%\(rcmail_config\['sendmail_delay'\] =\) .*$%\1 20;%" -i $MAINCFG
    sed "s%\(rcmail_config\['http_received_header'\] =\) .*$%\1 true;%" -i $MAINCFG
    
    ## USER INTERFACE
    sed "s%\(rcmail_config\['list_cols'\] = array\).*$%\1('flag', 'status', 'attachment', 'subject', 'from', 'date', 'size');%" -i $MAINCFG
    sed "s%\(rcmail_config\['max_pagesize'\] =\) .*$%\1 6000;%" -i $MAINCFG
    
    ## USER PREFERENCES
    sed "s%\(rcmail_config\['pagesize'\] =\) .*$%\1 600;%" -i $MAINCFG
    sed "s%\(rcmail_config\['show_images'\] =\) .*$%\1 1;%" -i $MAINCFG
    sed "s%\(rcmail_config\['htmleditor'\] =\) .*$%\1 2;%" -i $MAINCFG
    sed "s%\(rcmail_config\['preview_pane'\] =\) .*$%\1 true;%" -i $MAINCFG
    sed "s%\(rcmail_config\['logout_expunge'\] =\) .*$%\1 true;%" -i $MAINCFG
    sed "s%\(rcmail_config\['mime_param_folding'\] =\) .*$%\1 0;%" -i $MAINCFG
    sed "s%\(rcmail_config\['skip_deleted'\] =\) .*$%\1 true;%" -i $MAINCFG
    sed "s%\(rcmail_config\['check_all_folders'\] =\) .*$%\1 true;%" -i $MAINCFG
    sed "s%\(rcmail_config\['autoexpand_threads'\] =\) .*$%\1 2;%" -i $MAINCFG
    sed "s%\(rcmail_config\['mdn_requests'\] =\) .*$%\1 2;%" -i $MAINCFG

If you did an upgrade, once you're sure that the above changes are OK replace the old config with the new one:

    HOSTu:~# mv -vf $MAINCFG /etc/roundcube/main.inc.php
    HOSTu:~# MAINCFG="/etc/roundcube/main.inc.php"      # for other conf

### Enable and configure «show_additional_headers» plugin

    PLUGIN="show_additional_headers"
    grep --color -w "$PLUGIN" $MAINCFG || \
      sed "s%\(rcmail_config\['plugins'\] = array(.*\));$%\1, '$PLUGIN');%" -i $MAINCFG

The above commands will just append the plugin name to the plugins list.

    cat >> $MAINCFG <<__EOF__
    
    # plugin: show_additional_headers
    \$rcmail_config['show_additional_headers'] = '';
    __EOF__
    
    sed "s%\(rcmail_config\['show_additional_headers'\] =\) .*$%\1 array('Reply-To', 'Followup-To', 'List-Id', 'Sender', 'X-Sender', 'Delivered-To', 'User-Agent', 'X-Mailer', 'Organization');%" -i $MAINCFG

### Enable and configure «managesieve» plugin

This setup was tested only on a Dovecot IMAP server with ManageSieve.

    PLUGIN="managesieve"
    grep --color -w "$PLUGIN" $MAINCFG || \
      sed "s%\(rcmail_config\['plugins'\] = array(.*\));$%\1, '$PLUGIN');%" -i $MAINCFG

The above commands will just append the plugin name to the plugins list.

    CFG="/etc/roundcube/plugins/managesieve/config.inc.php"
    SUFFIX=".dist"
    
    rm -vf $CFG
    cp -va /usr/share/roundcube/plugins/managesieve/config.inc.php.dist $CFG$SUFFIX
    
    sed "s%\(rcmail_config\['managesieve_port'\] =\) .*$%\1 4190;%" -i $CFG$SUFFIX
    sed "s%\(rcmail_config\['managesieve_auth_type'\] =\) .*$%\1 'PLAIN';%" -i $CFG$SUFFIX
    sed "s%\($rcmail_config\['managesieve_script_name'\] =\) .*$%\1 'roundcube';%" -i $CFG$SUFFIX
    
    mv -vf $CFG$SUFFIX $CFG

### Enable and configure «squirrelmail_usercopy» plugin

This plugin is useful only if you migrate from `squirrelmail` to `roundcube` — automatic import some of users' preferences and the addressbook into Roundcube database on first login.

    PLUGIN="squirrelmail_usercopy"
    grep --color -w "$PLUGIN" $MAINCFG || \
      sed "s%\(rcmail_config\['plugins'\] = array(.*\));$%\1, '$PLUGIN');%" -i $MAINCFG

The above commands will just append the plugin name to the plugins list.

    chmod ug+r /var/lib/squirrelmail/data
    cat >> /etc/roundcube/plugins/squirrelmail_usercopy/config.inc.php <<__EOF__
    
    // Driver - 'file' or 'sql'
    \$rcmail_config['squirrelmail_driver'] = 'file';
    
    // full path to the squirrelmail data directory
    \$rcmail_config['squirrelmail_data_dir'] = '/var/lib/squirrelmail/data';
    \$rcmail_config['squirrelmail_data_dir_hash_level'] = 0;
    
    // identities_level option value for squirrelmail plugin
    // With this you can bypass/change identities_level checks
    // for operations inside this plugin. See #1486773
    \$rcmail_config['squirrelmail_identities_level'] = null;
    
    // set to false if your squirrelmail config setting $edit_identity has been true
    \$rcmail_config['squirrelmail_set_alias'] = false;
    __EOF__

## Add a virtual web site for *webmail* that contains:

    <VirtualHost *:80>
    	ServerAdmin	root@YOUR.domain
    	ServerName	mail.YOUR.domain
    	ServerAlias	mail
    	RedirectMatch ^/(.*)	https://mail.YOUR.domain/$1
    </VirtualHost>
    
    <VirtualHost mail.YOUR.domain:443>
    	ServerAdmin	root@YOUR.domain
    	ServerName	mail.YOUR.domain
    	RedirectMatch ^/$	https://mail.YOUR.domain/roundcube/
    	RedirectMatch ^/roundcube/..*/$	https://mail.YOUR.domain/roundcube/
    
    	SSLEngine		On
    	SSLCertificateFile	/etc/ssl/certs/mail.YOUR.domain.crt
    	SSLCertificateKeyFile	/etc/ssl/private/mail.YOUR.domain.key
    	SSLCACertificateFile	/usr/share/ca-certificates/cacert.org/cacert.org.crt
    
    	DocumentRoot /var/www/mail.YOUR.domain/
    	<Directory "/var/www/mail.YOUR.domain/">
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
    
    	CustomLog ${APACHE_LOG_DIR}/access_mail-YOUR-domain.log combined
    	ServerSignature On
    </VirtualHost>

Enable the webmail virtual host and restart _apache2_:

    HOSTu:~# a2ensite mail.YOUR.domain
    
    HOSTu:~# service apache2 restart


Known facts
-----------

### W: Assigning the return value of new by reference is deprecated in MDB2.php

These lines are constantly appearing in roundcube logs for each visitor:

    [..date..] PHP Deprecated:  Assigning the return value of
      new by reference is deprecated in /usr/share/php/MDB2.php on line 393
    [..date..] PHP Deprecated:  Assigning the return value of
      new by reference is deprecated in /usr/share/php/MDB2.php on line 2647

This was a bug in `php-mdb2` [version 2.5.0b2-1][11] fixed in Debian 7.0.

A minimal fix for this problem is to execute these commands:

    cp -va /usr/share/php/MDB2.php /usr/share/php/MDB2.php_2.5.0b2-1
    
    grep 'function &_wrapResult($result,' /usr/share/php/MDB2.php
    sed 's%\(function &_wrapResult($result\),%\1_resource,%' -i /usr/share/php/MDB2.php
    sed 's%\(reverse->tableInfo($result\));%\1_resource);%' -i /usr/share/php/MDB2.php
    sed 's%\($result =& new $class_name($this, $result\),%\1_resource,%' -i /usr/share/php/MDB2.php
    sed 's%\($result = new $result_wrap_class($result\),%\1_resource,%' -i /usr/share/php/MDB2.php
    
    grep '=& new' /usr/share/php/MDB2.php
    sed 's%db =& new%db = new%' -i /usr/share/php/MDB2.php
    sed 's%result =& new%result = new%' -i /usr/share/php/MDB2.php

If something is not working just restore the original file from backup.

[11]: http://bugs.debian.org/571702
