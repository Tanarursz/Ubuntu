# Ubuntu Szerver Alapjai

##  Betöltési folyamatok és Boot Manager

Ubuntu szerver indításakor a következő lépések történnek:

- **BIOS/UEFI**: Hardver inicializálása és a bootloader betöltése.
- **GRUB (GRand Unified Bootloader)**: Kiválasztja és betölti a Linux kernelt.
- **Init rendszer (systemd)**: A boot folyamat végrehajtása és a rendszer komponenseinek elindítása.

### GRUB konfigurációja

```bash
sudo nano /etc/default/grub
```

Konfiguráció példa:

```bash
GRUB_DEFAULT=0
GRUB_TIMEOUT=5
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
GRUB_CMDLINE_LINUX=""
```

A módosítások után frissítés:

```bash
sudo update-grub
```

##  Futási szintek (Runlevels)

Ubuntu a **systemd** rendszert használja, amely célokat (targets) kezel:

- `multi-user.target` (hálózati és többfelhasználós mód)
- `graphical.target` (grafikus felület, ha van)
- `rescue.target` (karbantartási mód)

### Aktuális futási szint megtekintése

```bash
systemctl get-default
```

### Futási szint megváltoztatása

```bash
sudo systemctl set-default multi-user.target
```

##  IP-cím beállítása a szerveren

Ubuntu szerveren az IP-cím beállítása a **Netplan** segítségével történik.

### Statikus IP-cím beállítása
Nyisd meg a Netplan konfigurációs fájlt:

__A netplan könyvtárban van egy .yaml végződésű fálj azt nyisd meg/!__

```bash
sudo nano /etc/netplan/XXX.yaml
```

Adja hozzá vagy módosítsa az alábbi beállításokat:

```yaml
network:
  version: 2
  ethernets:
    enp0s03:
      dhcp4: true
    enp0s08:
      dhcp4: false
      addresses:
        - 192.168.1.69/24
      routes:
        - to: 0.0.0.0/0
          via: 192.168.1.1
```
to: 0.0.0.0/0 → ez jelenti a default route-ot (minden forgalom ide megy, ha nincs más útvonal)

via: 192.168.1.1 → ez a gateway IP címe, amin keresztül az alapértelmezett útvonal halad

__Tehát ez a két sor együtt adja meg az alapértelmezett átjárót!__


Konfiguráció elmentése:

```bash
sudo netplan apply
```

Ha hiba lép fel:

```bash
sudo netplan --debug apply
```


### DHCP visszaállítása
Ha dinamikus IP-címet szeretnél visszaállítani, szerkeszd a fájlt és állítsd be:

```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: yes
```

Majd alkalmazd:

```bash
sudo netplan apply
```

Ez biztosítja, hogy az Ubuntu szerver mindig a megfelelő IP-címen legyen elérhető.

### IP-cím ellenőrzése
Az új beállítások ellenőrzéséhez használd:

```bash
ip a
```

Vagy:

```bash
ip route show
```
## Particionálás, fájlrendszerek és fájlműveletek

### Új lemez particionálása

```bash
sudo fdisk /dev/sdb
```
### Merevlemezek listázása

```bash
sudo fdisk -l
```

Parancsok:

- `n` - Új partíció létrehozása
- `p` - Elsődleges partíció
- `w` - Módosítások mentése



### Fájlrendszer létrehozása

```bash
sudo mkfs.ext4 /dev/sdb1
```

### Csatolás

```bash
sudo mount /dev/sdb1 /mnt
```

### Állandó csatolás (`/etc/fstab` szerkesztése)

```bash
/dev/sdb1  /mnt  ext4  defaults  0  2
```




##  Linkek kezelése Linux fájlrendszerben

A Linux fájlrendszerében kétféle link létezik:
- **Hard link**: Ugyanazon fájlt mutatja, nem különböztethető meg az eredetitől.
- **Symbolic (soft) link**: Egy hivatkozás egy másik fájlra vagy könyvtárra.

### Hard link létrehozása

```bash
ln eredeti_fájl link_neve
```
```bash
ln eredeti_fajl.txt hard_link_fajl.txt```
```

### Symbolic link létrehozása

```bash
ln -s eredeti_fájl link_neve
```

### Linkek listázása

```bash
ls -l
```

A szimbolikus linkek egy "->" jellel mutatnak az eredeti fájlra.

### Link törlése

- Hard link törlése:
```bash
rm link_neve
```

- Symbolic link törlése:
```bash
rm link_neve
```


## Fájlhozzáférések és ACL-ek

### Fájlengedélyek módosítása

```bash
sudo chmod 755 fájl_neve
```

### Tulajdonos módosítása

```bash
sudo chown user:group fájl_neve
```

### ACL használata

```bash
sudo setfacl -m u:felhasználó:rwx fájl_neve
```

##  DHCP és DNS szolgáltatások 

### DHCP szerver telepítése és beállítása
||| __javitott verzió__ |||

Telepítés:

```bash
sudo apt install isc-dhcp-server
```

Konfiguráld a következő fáljt (`/etc/dhcp/dhcpd.conf`): (*A tabulátor egy jó cucc, használd*)

```bash
sudo nano /etc/dhcp/dhcpd.conf
```
__kommenteld ki a fálj elején a következő sorokat: (rakj a kód elé #-et)__
```bash
#option domain-name "example.org";
#option domain-name-servers ns1.example.org, ns2.example.org;
```
__Ird bele a következőt:__

### Az "option routers"-nél azt az ip cimet add meg amit adtál statikus ip cimet a háló kártyának!

```bash

subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.100 192.168.1.200;
    option domain-name-servers 8.8.8.8, 8.8.4.4;
    option routers 192.168.1.69;
    default-lease-time 600;
    max-lease-time 7200;
}
```

dhcp szerver hálózati kártyájának beállitása:

```bash
sudo nano /etc/default/isc-dhcp-server
```
Konfiguráció (`/etc/default/isc-dhcp-server`):

## szerkezsd az alábbi fáljt e módon

```bash
INTERFACESv4="enp0s08"
INTERFACESv6=""
```

## tűzfal UFW || portok engedélyezése amik akadályozhatnák a DHCP működését

```bash

sudo ufw allow 68/udp
sudo ufw allow 67/udp
sudo ufw enable

```

dhcp szerver elinditása

```bash
sudo systemctl enable isc-dhcp-server
sudo systemctl start isc-dhcp-server
```
dhcp szerver ellenörzése

```bash
sudo systemctl status isc-dhcp-server
```
DHCP kész __:3__ 	     __(づ ◕‿◕ )づ__    

Konfiguráció (`/etc/bind/named.conf.local`):

```bash
zone "example.com" {
    type master;
    file "/etc/bind/db.example.com";
```

##  Web- és adatbázis-kiszolgálók

### Apache telepítése és beállítása

```bash
sudo apt install apache2
```

Konfiguráció (`/etc/apache2/sites-available/000-default.conf`):

```bash
<VirtualHost *:80>
    DocumentRoot /var/www/html
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Újrakonfigurálás:

```bash
sudo systemctl restart apache2
```

### MySQL telepítése és beállítása

```bash
sudo apt install mysql-server
```

Biztonsági beállítások:

```bash
sudo mysql_secure_installation
```

##  Forgalomirányítás és címfordítás (NAT)

A hálózati csomagok irányításához és címfordításához az `iptables` vagy a `nftables` eszközök használhatók.

###  IP forwarding engedélyezése

Az IP-csomagok továbbításának engedélyezése:

```bash
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
```



##  Tűzfal és Proxy beállítások

### UFW tűzfal beállítása

```bash
sudo ufw enable
sudo ufw allow 80/tcp
```

### Squid proxy telepítése és konfigurálása

```bash
sudo apt install squid
```

Konfiguráció (`/etc/squid/squid.conf`):

```bash
acl mynet src 192.168.1.0/24
http_access allow mynet
http_port 3128
```

## Shell-beállítások, segédprogramok és pipeline

### Shell környezeti változók beállítása

```bash
export PATH=$PATH:/custom/path
```

### Pipeline használata

```bash
ls -l | grep .txt | wc -l
```
##  Levelezési szolgáltatások

### Postfix telepítése és konfigurálása

```bash
sudo apt install postfix
```

Beállítás (`/etc/postfix/main.cf`):

```bash
myhostname = mail.example.com
mydestination = example.com, localhost.localdomain, localhost
relayhost = 
mailbox_size_limit = 0
recipient_delimiter = +
inbox_style = Maildir
```

### Dovecot telepítése és konfigurálása

```bash
sudo apt install dovecot-imapd dovecot-pop3d
```

Konfiguráció (`/etc/dovecot/dovecot.conf`):

```bash
protocols = imap pop3
mail_location = maildir:~/Maildir
```
