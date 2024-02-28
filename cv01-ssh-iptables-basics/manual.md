# Cviceni 1

## Zakladni prikazy
To máme umět ze ZOS.
```bash
ps auxf
ls | ls -1 | ls -al
mkdir | mkdir -p
touch | chmod | chown
man | info
cd | cd - | cd ~ | pwd
netstat | netstat -tupln
w | who | id
find
grep | grep -R
cat | sort | mc |cut
nano | vim | for | while | if | read
sed | awk
```

## SSH

### Autentizace klici

#### Vytvoření a přidání klíče
- dobré je nezapomenout zadané heslo jako cvíčící na druhém cvičení nebo nezadávat žádné heslo
```bash
ssh-keygen -t rsa -b 4096 -C "jindra@spos"
ssh-add
```
#### Přidání do agenta
- nemusíme zadávat při každém připojení heslo ke klíči
- umožňuje přenášet klíče skrz servery
```bash
ssh-agent || eval `ssh-agent`
```

#### Přidání klíče na server
```bash
ssh-copy-id user@host
```
### Authorized keys

```bash
cat ~/.ssh/authorized_keys
command="./run-this",no-port-forwarding,no-x11-forwarding,no-agent-forwarding ssh-dss KEY user@machine
```

### Prihlaseni 

```bash
ssh -i ~/.ssh/id_dsa  -l user machine
ssh -l user machine.local
ssh user@machine.local
ssh user@machine "uname -a"
```

### Nastaveni klienta
- umožňuje přednastavení připojení
```
nano ~/.ssh/config

Host spos
   HostName 147.228.67.42
   User root
   Port 22
```
```bash
ssh spos
```

### Zakladni nastaveni serveru
- sshd = SSH Daemon
- nastavení chování serveru
```bash
nano /etc/ssh/sshd_config

...
AllowUsers root
PasswordAuthentication no
PermitRootLogin without-password
...

service sshd restart || systemctl restart sshd
```

## Úkol, který jsme neměli, ale cvičící to popírá
- změna hesla roota
```bash
passwd root
```

## Git
```bash
cd /etc
git init .
git add .
git commit -am "Prvni cviceni"
git diff
git log -p
```

## Informace o serveru a nastaveni:

```bash
/etc/
/etc/resolv.conf
/etc/hostname
/etc/network/interfaces
# připojitelné file systémy, umí i síťové disky
/etc/fstab
# statické záznamy DNS
/etc/hosts
/etc/passwd
/etc/group
# hashe hesel i doba jejich platnosti
/etc/shadow
# číslo verze systému
/etc/debian_version | /etc/redhat-release

# napíše hostname
hostname
# změní v runtime hostname na stroj, po restartu ze změní na hodnotu z /etc/hostname
hostname stroj
# informace o systému
uname -a
# čas od spuštění a průměrná zátěž
uptime
# výpis blokových zařízení
lsblk
# vypíše všechno aktuálně mountované (třeba i snapy)
mount || /proc/mounts
iptables
ifconfig | route | ip
dpkg | aptitude

free
cat /proc/cpuinfo | cat /proc/meminfo
mount
netstat -tupln
crontab -l
# správci úloh
top
htop
```

## Iptables
 
```bash
# omezene pristupu na SSH jen z univerzity
# informace o pravidlech firewallu
iptables-nvL
# přidá na konec pravidel následující pravidla (pouze pro runtime)
iptables -A INPUT -s  147.228.0.0/16  -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -p tcp --dport 22  -j DROP
# auto policy nastaví pro INPUT na DROP
iptables -P INPUT DROP
# konfigurace stavového modulu
iptables -I INPUT -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT
iptables -I OUTPUT -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT

# ulozeni a obnoveni nastaveni
iptables-save > /etc/network/iptables
iptables-restore /etc/network/iptables

# nastaveni po nahozeni interface
nano /etc/network/interfaces
	post-up /sbin/iptables-restore /etc/network/iptables

```

## Fail2ban

```bash
apt-get install fail2ban

cd /etc/fail2ban/
nano jail.conf

```

## TCPWrappers

```bash
# overeni podpory
ldd /usr/sbin/sshd | grep libwrap

# nastaveni povolenych/zakazanych hostu
/etc/hosts.allow
	sshd : 147.228.0.0/255.255.0.0
/etc/hosts.deny

# kontrola nastaveni
tcpdmatch sshd localhost
tcpdchk
```

## Sprava sluzeb

```bash
# start | stop | restart | status | ...
service fail2ban restart
/etc/init.d/fail2ban restart
systemctl restart fail2ban.service
```
