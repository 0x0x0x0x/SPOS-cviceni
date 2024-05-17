# Opakování před testem

## Rozsypaný server
- třeba budou chybět editory a nebude fungovat síť

echo "" > file.txt
cat > file.txt

sed -i 's/replace/with/' file.txt

kontrolovat dns

mount -o remount,rw /dev/vda
## Zabezpečení (1 bod)

- přidat klíč, zrušit přihlášení heslem

iptables, fail2ban

## LVM

LVM z volných disků, VG=data, LV=data base, velikost = 2GB, format = ext4, připojit po startu do /srv/database

lsblk

vgcreate data /dev/vdb[1,2] /dev/vdc[1,2]

## POSTGRES
uzivatel spos, heslo heslo123., db spos, tabulka zavody - id, name a nastvate moznost pripojeni pouze z localhostu, změna portu

pod uživatlem postgres se přistoupí do databáze

## DNS

vytvořte DNS pro doménu test.spos a vytvorte zaznamy pro www, postu odkazujici na vasi verejnou ip

## Mail

nastavte postu pro domenu test.spos, maily pro adresu kancelar@test.spos jdou do maildiru uzivatele pepa

nastavte poštu pro domeny karel.spos a pepa.spos a zajistete, že info@karel.spos půjde uzivateli karel a info@pepa.spos pujde uživateli pepa

-- virtual domains nebude

## Webserver

nainstalujte webserver s podporou PHP a vytvorte stranku, ktera zobrazi obsah db z bodu 3

## Dalsi webserver

zpristupnete z predchoziho bodu pres https pomoci nginx