# Cviceni 10

## DNS round robin

Vice stejnych A zaznamu

```
nano db.akriz.zcu.spos

www	IN	A	147.228.67.42
www	IN	A	147.228.67.43
www	IN	A	147.228.67.44

service bind9 restart
host www.akriz.zcu.spos 127.0.0.1
```

## Rsync

```
ip a a 10.0.0.44/32 dev ens18
ip a a 10.0.1.44/32 dev ens18
ip a a 10.0.2.44/32 dev ens18
ip a a 10.0.3.44/32 dev ens18

rsync -rav /var/www/jindra.spos  /var/www/jindra.spos
```

## Pound
nepoužívá se - přeskočeno :(
```
apt-get install pound
```
```
/etc/pound/pound.cfg

	Service
		HeadRequire "Host:.*www.jindra.spos.*"
		BackEnd
			Address	127.0.0.1
			Port	80
			Priority 5
		End
		BackEnd
			Address 10.228.67.42
			Port    80
			Priority 5
		End
	End
```

```bash
poundctl -c /var/run/pound/poundctl.socket
poundctl -c /var/run/pound/poundctl.socket -b 0 0 1
```

## NginX

```
apt-get install nginx
```

```
nano /etc/apache2/ports.conf
na 8080 a 8443



#/etc/nginx/sites-available/lb.conf

upstream backend  {
        ip_hash;
        server www1:8080 max_fails=2 fail_timeout=5s;
        server www2:8080 max_fails=2 fail_timeout=5s;
}

server {
        listen 80;
        server_name www.jindra.spos;

        location / {
                proxy_pass http://backend;
                include proxy.include;
        }
}
```

```
#proxy.include

    proxy_set_header   Host $http_host;
    proxy_set_header   X-Real-IP $remote_addr;
    proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header   X-Forwarded-Proto $scheme;
    proxy_connect_timeout      90;
    proxy_send_timeout         300;
    proxy_read_timeout         300;
```

### HAProxy

<https://www.haproxy.com/>

### Traefik

<https://docs.traefik.io/>

### Vice Apache na jednom serveru

**Debian-way**

```
less  /usr/share/doc/apache2/README.multiple-insances
/usr/share/doc/apache2/examples/setup-instance oldphp
```

**Docker-way**

Docker + vice instanci Apache na ruznych portech a pred tim Nginx na rozhazovani zateze.
