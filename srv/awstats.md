awstats :: 6.9.5~dfsg-5 (web)
=============================

[Advanced Web Statistics][1] — `AWStats` — is a powerful web server logfile analyzer written in perl that shows you all your web statistics including visits, unique visitors, pages, hits, rush hours, search engines, keywords used to find your site, robots, broken links and more.

[1]: http://awstats.sourceforge.net


Install software packages
-------------------------

Install a recommended set of **awstats** related packages:

    HOSTu:~# apt install awstats libnet-dns-perl libnet-ip-perl \
               libgeo-ipfree-perl libgeo-ip-perl
    
    The following extra packages will be installed:
      libdigest-hmac-perl libdigest-sha1-perl

Note: **apt** is an alias for `apt-get` but you can use `aptitude` as well.


Configure AWStats service
-------------------------

Disable default _awstats_ example configuration and move data to `/srv/awstats`.

    HOSTu:~# cd /etc/awstats/ && mkdir -v conf.d
    HOSTu:~# mv -v awstats.conf awstats.conf_6.9.5~dfsg-5
    HOSTu:~# sed "s%\sawstats$%%" -i /usr/share/awstats/tools/update.sh
    
    HOSTu:~# mv -v /var/lib/awstats /srv/ && ln -s /srv/awstats /var/lib/awstats

Disable automatic build of _awstats_ static pages.

    HOSTu:~# sed 's%^\(AWSTATS_ENABLE_BUILDSTATICPAGES\)=.*$%\1="no"%' -i /etc/default/awstats

Add a virtual web site for _awstats_ service that contains:

    	Alias /awstats-icon/ "/usr/share/awstats/icon/"
    	<Directory "/usr/share/awstats/icon/">
    		AllowOverride	None
    		Options	FollowSymLinks MultiViews
    	</Directory>
    
    	ScriptAlias /cgi-bin/ /usr/lib/cgi-bin/
    	<Directory "/usr/lib/cgi-bin">
    		AllowOverride	None
    		Options	+ExecCGI -MultiViews +SymLinksIfOwnerMatch
    	</Directory>

Configure _apache2_ to allow user `www-data` access to all logs:

    HOSTu:~# chmod o+x /var/log/apache2
    HOSTu:~# chmod a+r /var/log/apache2/access_*log
    
    HOSTu:~# sed "s%create 640%create 644%" -i /etc/logrotate.d/apache2

### Add a new web site to AWStats monitoring

    HOSTu:~# set -eu
    WEBSITE="VHOST.example.com"
    
    CFGFILE="/etc/awstats/conf.d/${WEBSITE//./-}.conf"
    
    cd /etc/awstats
    cp -va awstats.conf_6.9.5~dfsg-5 $CFGFILE
    ln -s "conf.d/${WEBSITE//./-}.conf" "awstats.${WEBSITE/%.*/}.conf"
    
    sed "s%^\(LogFile=\"/var/log/apache2/access\).*$%\1_${WEBSITE//./-}.log\"%" -i $CFGFILE
    sed 's%^LogFormat=.*$%LogFormat=1%' -i $CFGFILE
    sed 's%^AllowFullYearView=.*$%AllowFullYearView=3%' -i $CFGFILE
    sed 's%^SkipHosts=.*$%SkipHosts="::1 127.0.0.1 REGEX[^10\\.]"%' -i $CFGFILE
    sed 's%^SkipFiles=.*$%SkipFiles="REGEX[^\/awstats]"%' -i $CFGFILE
    
    sed 's%^\(LevelForWormsDetection\)=[0-9]*%\1=2%' -i $CFGFILE
    sed 's%^UseFramesWhenCGI=.*$%UseFramesWhenCGI=0%' -i $CFGFILE
    sed 's%^ShowWormsStats=.*$%ShowWormsStats=HBL%' -i $CFGFILE
    sed 's%^ShowMiscStats=.*$%ShowMiscStats=anjdfrqwp%' -i $CFGFILE
    
    sed 's%^#*\(LoadPlugin="geoip\)%#\1%' -i $CFGFILE
    sed 's%^#*\(LoadPlugin="geoipfree\)%\1%' -i $CFGFILE
    sed 's%^#*\(LoadPlugin="hostinfo\)%\1%' -i $CFGFILE
    sed 's%^#*\(LoadPlugin="rawlog\)%\1%' -i $CFGFILE
    
    sed 's%^\(HostAliases=".*\)"$%\1 www REGEX[example.com]"%' -i $CFGFILE
    sed "s%^\(SiteDomain=\"\).*$%\1${WEBSITE}\"%" -i $CFGFILE

Initialize and run manually the first _awstats_ update for this web site:

    su www-data -c "/usr/lib/cgi-bin/awstats.pl  --init -config=VHOST"
    su www-data -c "/usr/lib/cgi-bin/awstats.pl -update -config=VHOST"

Check if the _awstats_ counters are incremented for „VHOST” web site:

    http://awstats.YOUR-DOMAIN/cgi-bin/awstats.pl?config=VHOST

### Add extra „# of hits” sections to AWStats monitoring

I want to customize _awstats_ to count how many files are downloaded from the web site, but separated in two categories: completed downloads and aborted downloads.

First, you need to configure _Apache_ web server to write this information in the logs:

    HOSTu:~# echo 'LogFormat "%h %l %u %t \"%r\" %>s %X %O \"%{Referer}i\" \"%{User-Agent}i\"" combinedX' \
      > /etc/apache2/conf.d/logformat-combinedX

This configuration just adds an extra parameter «%X» to the standard _Apache_ „combined” log format, but under a different name: `combinedX`. Modify the virtual host monitored with _awstats_ to use this new log format and restart `apache2` service:

    HOSTu:~# vi /etc/apache2/sites-available/VHOST.example.com
    [..]
    CustomLog ${APACHE_LOG_DIR}/access_VHOST-example-com.log combinedX

Update the «LogFormat» parameter in the _awstats_ config file for „VHOST” web site by extending the _Apache_ „combined” format (also present in the config file, on the same section, as an example):

    HOSTu:~# vi /etc/awstats/awstats.VHOST.conf
    [..]
    LogFormat = "[..] %code %extra1 %bytesd [..]"

The last step is to add these two extra sections in the _awstats_ config file for „VHOST” web site:

    ExtraSectionName1="PRODUCT Download Hits"
    ExtraSectionCodeFilter1="200 304"
    ExtraSectionCondition1="extra1,[-+]"
    ExtraSectionFirstColumnTitle1="PRODUCT completed downloads"
    ExtraSectionFirstColumnValues1="URL,^(\/pub\/1.0\/PRODUCT.*)$"
    ExtraSectionFirstColumnFormat1="%s"
    ExtraSectionStatTypes1=HBL
    ExtraSectionAddSumRow1=1
    MaxNbOfExtra1=20
    MinHitExtra1=1
    
    ExtraSectionName2="PRODUCT Download Hits"
    ExtraSectionCodeFilter2="200 206 304"
    ExtraSectionCondition2="extra1,X"
    ExtraSectionFirstColumnTitle2="PRODUCT aborted downloads"
    ExtraSectionFirstColumnValues2="URL,^(\/pub\/1.0\/PRODUCT.*)$"
    ExtraSectionFirstColumnFormat2="%s"
    ExtraSectionStatTypes2=HBL
    ExtraSectionAddSumRow2=1
    MaxNbOfExtra2=20
    MinHitExtra2=1

Two new „PRODUCT Download Hits” sections will appear in your AWStats interface.


Known facts
-----------

### Cannot load plugin "robots"

On _awstats_ interface there is no robot or worm detected. Trying to load the plugins will show this message:

    Error: AWStats config file contains a directive to load plugin "robots"
      (LoadPlugin="robots") but AWStats can't open plugin file "robots.pm"
      for read. Check if file is in "/usr/lib/cgi-bin/plugins" directory and
      is readable.
    
    Setup ('/etc/awstats/awstats.HOST.conf' file, web server or permissions)
      may be wrong. Check config file, permissions and AWStats documentation
      (in 'docs' directory).

The error comes from the fact that the file 'robots.pm' is not valid for a plugin, it can only specify a list of known web robots. Also, it is localized in the 'lib' directory instead of the 'plugins' directory  were all the plugins are located.
