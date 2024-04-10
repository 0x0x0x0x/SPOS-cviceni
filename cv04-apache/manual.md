# Cviceni 4

## Apache2

```bash
apt-get install apache2
```

### Nastaveni

```bash
cd /etc/apache2

a2enmod | a2dismod
a2ensite | a2dissite

user: www-data / httpd

service apache2 reload

/etc/apache2/sites-*
/etc/apache2/mods-*
/etc/apache/ports.conf

apachectl -t | apachectl -S
```

### Data a logy
```
/var/www/

/var/log/apache2/*access.log
/var/log/apache2/*error.log
 ```

### Virtualni server

```bash
cd /etc/apache2/sites-avaible/
cp 000-default.conf akriz.zcu.spos.conf
nano akriz.zcu.spos.conf

<VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        ServerName akriz.zcu.spos
        ServerAlias www.akriz.zcu.spos

        ServerAdmin webmaster@akriz.zcu.spos
        DocumentRoot /var/www/akriz.zcu.spos

        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog ${APACHE_LOG_DIR}/akriz.zcu.spos.error.log
        CustomLog ${APACHE_LOG_DIR}/akriz.zcu.spos.access.log combined

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        #Include conf-available/serve-cgi-bin.conf
</VirtualHost>

mkdir -p /var/www/akriz.zcu.spos
echo "akriz.zcu.spos" > /var/www/akriz.zcu.spos/index.html
a2ensite akriz.zcu.spos
# kontrola syntaxe
apachectl -t
apachectl -S
# restart
systemctl restart apache2
# test
curl -H 'Host: akriz.zcu.spos' http://localhost
curl -H 'Host: www.akriz.zcu.spos' http://localhost

nano /etc/hosts
127.0.0.1 akriz.zcu.spos www.akriz.zcu.cz
curl -v www.akriz.zcu.spos

a2enmod ssl
a2ensite default-ssl

cd sites-enabled/
cp default-ssl.conf  akriz.zcu.spos.conf
nano akriz.zcu.spos.conf
...
ServerAdmin webmaster@localhost
ServerName akriz.zcu.spos
ServerAlias www.akriz.zcu.spos
DocumentRoot /var/www/akriz.zcu.spos
...
systemctl restart apache2
curl -v -k https://akriz.zcu.spos


// ukázka komplexního nastavení
<VirtualHost *:80>
        ServerAdmin admin@jindra.spos
        ServerName www.jindra.spos
        ServerAlias web.jindra.spos

        DocumentRoot /var/www/www.jindra.spos

        Redirect / https://www.jindra.spos/

        Alias /pma /usr/share/phpmyadmin
        
        <Directory />
                Options FollowSymLinks

                AllowOverride None
        </Directory>

        ProxyPass /app1 http://localhost:3333/
        ProxyPassReverse /app1 http://localhost:3333/

        <Directory /var/www/>
                Options Indexes FollowSymLinks MultiViews
                AllowOverride All
                Order allow,deny
                allow from all
        </Directory>

        <Location /wsNS_1.0>
                AuthType Basic
                AuthName "Restricted Access"
                AuthUserFile /etc/apache2/.htpasswd
                Require user app_user1
        </Location>

        # Possible values include: debug, info, notice, warn, error, crit,
        # alert, emerg.
        LogLevel warn

        CustomLog ${APACHE_LOG_DIR}/www.jindra.spos.access.log combined
        ErrorLog ${APACHE_LOG_DIR}/www.jindra.spos.error.log

</VirtualHost>
```

## Dehydrated
```bash
nano /etc/dehydrated/domains.txt
 ...
 CONTACT_EMAIL=akriz@students.zcu.cz


```

## PHP 5/7

```bash
apt-get install libapache2-mod-php

#/etc/php5/apache2
/etc/php/7.4/apache2/php.ini
#/etc/php5/cli
```

### Nastaveni

```bash
upload_max_filesize = 16M
memory_limit = 128M
max_input_time = 60
post_max_size = 16M
max_execution_time = 30

open_basedir = ...
disable_functions = ...
disable_classes = ...
allow_url_fopen = On/Off
allow_url_include = On/Off
display_errors = On/Off
error_reporting = E_ALL
log_errors = On/Off
```

### htacess & vhost

```
php_admin_flag safe_mode On
php_admin_value open_basedir /var/www/www.jindra.spos
php_admin_value upload_tmp_dir /var/www/www.jindra.spos/phptmp
php_admin_value session.save_path /var/www/www.jindra.spos/phptmp
```

### Informace

```php
<?php
  phpinfo();
?>
```

### Bezpecnost

```php
// http://jindra.spos/index.php?page=/etc/passwd

<?php
    $page = $_GET['page'];
    $xfile = fopen($page, "r") or die("Unable to open file!");
    echo fgets($xfile);
    fclose($xfile);
?>
```


### HTTPS/ SSL

**Self signed certifikat**

```bash
openssl genrsa -des3 -out server.key 1024
openssl req -new -key server.key -out server.csr
cp server.key server.key.org
openssl rsa -in server.key.org -out server.key
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```

**Lets encrypt certifikat**

```bash
cd /opt && git clone https://github.com/lukas2511/dehydrated && cd dehydrated
cp ./docs/examples/hook.sh /etc/dehydrated/hook.sh && chmod +x /etc/dehydrated/hook.sh
vim /etc/dehydrated/config
dehydrated --register --accept-terms
```

```
DOMAINS_TXT=/etc/dehydrated/domains
WELLKNOWN="/var/www/dehydrated/.well-known/acme-challenge/"
CERTDIR="/etc/letsencrypt/live/"
CONTACT_EMAIL=skupaj@students.zcu.cz
HOOK=/etc/dehydrated/hook.sh
```

Apache config

```
	Alias /.well-known/acme-challenge/ /var/www/dehydrated/.well-known/acme-challenge/
```

Nginx config

```
        location /.well-known/acme-challenge {
            alias   /var/www/dehydrated/.well-known/acme-challenge;
        }
```
