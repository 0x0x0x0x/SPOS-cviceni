iptables -nvL
lsblk


nano /etc/dovecot/conf.d/10-mail.conf



# WEBSERVER + PHP

## LOCAL DNS

```sh
nano /etc/hosts
```
add:
```config
127.0.0.2       test.spos
```

## APACHE

```sh
apt install apache2 php

cd /etc/apache2/sites-avaliable
cp 000-default.conf test.spos.conf

a2ensite test.spos
#a2dissite 

echo "Page not found" > /var/www/html/index.html

nano test.spos.conf
```
Examples:

```xml
<VirtualHost 10.0.0.15:8080>
        ServerName test.spos
        DocumentRoot /var/www/test8080
</VirtualHost>

<VirtualHost *:80>
        ServerName test.spos
        DocumentRoot /var/www/test8080
</VirtualHost>
```

```sh

nano /etc/apache2/ports.conf
Listen 10.0.0.15:8888
Listen 10.0.0.15:8080


service apache2 restart
```


## NGINX

```sh
apt install nginx php php-fpm

cd /etc/nginx/sites-available

cp default test.spos

ln -s /etc/nginx/sites-available/test.spos /etc/nginx/sites-enabled/

echo "Page not found" > /var/www/html/index.html

nano test.spos
```
viz https://www.theserverside.com/blog/Coffee-Talk-Java-News-Stories-and-Opinions/Nginx-PHP-FPM-config-example

Virtuální servery bez php:
```
server {
        listen 10.0.0.15:8181;
        server_name test.spos;
        root /var/www/web8181;
        index index.html;
}

server {
        listen 8282;
        server_name test.spos;
        root /var/www/web8282;
        index index.html;
}
```



# BALANCER

## NGINX

```
upstream balancer {
        server localhost:8181;
        server localhost:8282;
}


server {
        listen 8080;
        server_name test.spos;
        location / {
                proxy_pass http://balancer;
                #proxy_set_header Host            $host;
                #proxy_set_header X-Forwarded-For $remote_addr;
        }

}
```

# SSL
```sh
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/server.key -out /etc/ssl/certs/server.crt

curl --insecure https://test.spos:443
```


## NGINX

```
server {
        listen 443 ssl;
        server_name test.spos;
        location / {
                proxy_pass http://balancer;
                proxy_set_header Host            $host;
                proxy_set_header X-Forwarded-For $remote_addr;
        }


        ssl on;
        ssl_certificate /etc/ssl/certs/server.crt;
        ssl_certificate_key /etc/ssl/private/server.key;
}
```



# Zákaz funkcí PHP

## APACHE
```sh
nano /etc/php/7.4/apache2/php.ini

service apache2 restart

```
## NGINX (FPM)
```sh
nano /etc/php/7.4/fpm/php.ini

service php7.4-fpm restart
```




# DATABASE

viz návod PDF

## PSQL
```sh
apt install php-pgsql
```

Tvorba tabulky
```sh

CREATE TABLE accounts (
  user_id SERIAL PRIMARY KEY, 
  username VARCHAR (50) UNIQUE NOT NULL, 
  password VARCHAR (50) NOT NULL, 
  email VARCHAR (255) UNIQUE NOT NULL, 
  created_at TIMESTAMP NOT NULL, 
  last_login TIMESTAMP
);

CREATE TABLE users (
  id SERIAL PRIMARY KEY, 
  firstname VARCHAR (50) NOT NULL, 
  email VARCHAR (255) NOT NULL,
  skore INTEGER NOT NULL, 
  created_at TIMESTAMP NOT NULL
);


for i in `seq 1 30`; 
do 
random_email=$(openssl rand -hex 8)@gmail.com
echo "INSERT INTO users VALUES(DEFAULT,'HONZA $i', '$random_email', $RANDOM , now());" |
psql spos; done

```




## MYSQL
```sh
apt install mariadb-server
apt install php-mysql
```

```sh
for i in `seq 1 30`; 
do 
random_email=$(openssl rand -hex 8)@gmail.com
echo "INSERT INTO users VALUES(DEFAULT,'HONZA $i', '$random_email', $RANDOM , now());" |
mysql -h10.0.0.15 -uspos -pspos*2018 spos; done

```





# PHP show table

## MYSQL
```php
<?php
    $conn = new mysqli("10.0.0.15","spos","spos*2018","spos","3306");
    $sql = "SELECT * FROM users ORDER BY skore DESC";
    $result = $conn->query($sql);
    if($result->num_rows > 0) {
        while($row = $result->fetch_assoc()) {
            echo $row["firstname"].",";
            echo $row["email"].", ";
            echo $row["skore"]." ";
            echo "<br>";
        }
    }
    $conn->close();
?>
```


## PGSQL
```php
<?php
    $conn = pg_connect("host=10.0.0.15 dbname=spos user=spos_data password=spos_data");
    $sql = "SELECT * FROM users ORDER BY skore DESC";
    $result = pg_query($conn, $sql);
    if(pg_num_rows($result) > 0) {
        while($row = pg_fetch_row($result)) {
            echo $row[1].",";
            echo $row[2].", ";
            echo $row[3]." ";
            echo "<br>";
        }
    }
    pg_close($conn);
?>
```
