# Ubuntu Szerver Alapjai

## 1. Betöltési folyamatok és Boot Manager

Ubuntu szerver indításakor a következő lépések történnek:

- **BIOS/UEFI**: Hardver inicializálása és a bootloader betöltése.
- **GRUB (GRand Unified Bootloader)**: Kiválasztja és betölti a Linux kernelt.
- **Init rendszer (systemd)**: A boot folyamat végrehajtása és a rendszer komponenseinek elindítása.

### GRUB konfigurációja

```bash
sudo nano /etc/default/grub
```

A módosítások után frissítés:

```bash
sudo update-grub
```

## 2. Futási szintek (Runlevels)

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

## 3. Particionálás, fájlrendszerek és fájlműveletek

### Új lemez particionálása

```bash
sudo fdisk /dev/sdb
```

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

## 4. Fájlhozzáférések és ACL-ek

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

## 5. Shell-beállítások, segédprogramok és pipeline

### Shell környezeti változók beállítása

```bash
export PATH=$PATH:/custom/path
```

### Pipeline használata

```bash
ls -l | grep .txt | wc -l
```

## 6. DHCP és DNS szolgáltatások

### DHCP szerver telepítése

```bash
sudo apt install isc-dhcp-server
```

Konfiguráció (`/etc/dhcp/dhcpd.conf` szerkesztése).

### DNS szerver (Bind9) telepítése

```bash
sudo apt install bind9
```

Konfiguráció: `/etc/bind/named.conf.local`

## 7. Forgalomirányítás és címfordítás (NAT)

### IP forwarding engedélyezése

```bash
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
```

### Masquerading beállítása

```bash
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

## 8. Web- és adatbázis-kiszolgálók

### Apache telepítése

```bash
sudo apt install apache2
```

### MySQL telepítése

```bash
sudo apt install mysql-server
```

### PHP telepítése

```bash
sudo apt install php libapache2-mod-php
```

## 9. Tűzfal és proxy beállítások

### UFW tűzfal beállítása

```bash
sudo ufw enable
sudo ufw allow 80/tcp
```

### Squid proxy telepítése

```bash
sudo apt install squid
```

Konfiguráció: `/etc/squid/squid.conf`

## 10. Shell-szkriptek

### Egyszerű shell script létrehozása

```bash
echo -e "#!/bin/bash
echo 'Hello, Ubuntu!'" > script.sh
chmod +x script.sh
./script.sh
```

## 11. Levelezési szolgáltatások

### Postfix telepítése

```bash
sudo apt install postfix
```

### Dovecot telepítése (IMAP/POP3 szerver)

```bash
sudo apt install dovecot-imapd dovecot-pop3d
```

Konfiguráció: `/etc/postfix/main.cf` és `/etc/dovecot/dovecot.conf`

Ez a dokumentum összefoglalja az Ubuntu szerver alapvető beállításait és szolgáltatásait.
