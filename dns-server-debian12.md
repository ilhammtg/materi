# Tutorial Konfigurasi DNS Server Debian 12

## Bagian 1: Konfigurasi IP Static

### 1.1 Cek Interface Network

```bash
# Login sebagai root
su -

# Cek nama interface network
ip addr
```

Catat nama interface (biasanya: `enp0s3`, `eth0`, atau `ens33`)

### 1.2 Konfigurasi IP Static

Edit file konfigurasi network:
```bash
nano /etc/network/interfaces
```

Tambahkan/ubah konfigurasi (contoh interface: `enp0s3`):
```
# Interface loopback
auto lo
iface lo inet loopback

# Interface network
auto enp0s3
iface enp0s3 inet static
    address 192.168.1.10
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 8.8.8.8
```

**Sesuaikan:**
- `enp0s3` → nama interface Anda
- `192.168.1.10` → IP yang diinginkan
- `192.168.1.1` → IP gateway router Anda

Simpan dengan `Ctrl+O`, Enter, lalu `Ctrl+X`

### 1.3 Restart Network

```bash
# Restart networking
systemctl restart networking

# Atau reboot sistem
reboot
```

Setelah restart, cek IP:
```bash
ip addr show enp0s3
```

### 1.4 Set Hostname

```bash
# Set hostname
hostnamectl set-hostname dns-server.example.local

# Edit hosts file
nano /etc/hosts
```

Isi file `/etc/hosts`:
```
127.0.0.1       localhost
192.168.1.10    dns-server.example.local dns-server

# IPv6
::1             localhost ip6-localhost ip6-loopback
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters
```

---

## Bagian 2: Update Sistem dan Install Tools

```bash
# Update repository
apt update

# Upgrade sistem
apt upgrade -y

# Install tools yang diperlukan
apt install -y nano net-tools dnsutils bind9-utils
```

---

## Bagian 3: Instalasi BIND9 (DNS Server)

### 3.1 Install BIND9

```bash
apt install -y bind9 bind9utils bind9-doc
```

### 3.2 Backup Konfigurasi Default

```bash
cp /etc/bind/named.conf.options /etc/bind/named.conf.options.backup
cp /etc/bind/named.conf.local /etc/bind/named.conf.local.backup
```

---

## Bagian 4: Konfigurasi DNS Server

### 4.1 Konfigurasi File Options

Edit file options:
```bash
nano /etc/bind/named.conf.options
```

Hapus semua isi dan ganti dengan:
```
options {
    directory "/var/cache/bind";
    
    // Allow queries from local network
    allow-query { localhost; 192.168.1.0/24; };
    
    // Allow recursion for local network
    allow-recursion { localhost; 192.168.1.0/24; };
    
    // Forwarders (DNS Google dan Cloudflare)
    forwarders {
        8.8.8.8;
        8.8.4.4;
        1.1.1.1;
    };
    
    // Enable recursion
    recursion yes;
    
    // Listen on IPv4
    listen-on { any; };
    
    // Disable IPv6 (opsional)
    listen-on-v6 { none; };
    
    // DNSSEC validation
    dnssec-validation auto;
};
```

**Sesuaikan** `192.168.1.0/24` dengan subnet jaringan Anda!

### 4.2 Konfigurasi Zone

Edit file zone:
```bash
nano /etc/bind/named.conf.local
```

Tambahkan konfigurasi zone forward dan reverse:
```
// Forward Zone
zone "example.local" {
    type master;
    file "/etc/bind/zones/db.example.local";
};

// Reverse Zone (untuk 192.168.1.x)
zone "1.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/zones/db.192.168.1";
};
```

**Catatan:** Ganti `example.local` dengan domain yang Anda inginkan

### 4.3 Buat Direktori Zone

```bash
mkdir -p /etc/bind/zones
```

### 4.4 Buat File Forward Zone

```bash
nano /etc/bind/zones/db.example.local
```

Isi dengan:
```
;
; BIND data file for example.local
;
$TTL    604800
@       IN      SOA     dns-server.example.local. admin.example.local. (
                              2024102901    ; Serial (YYYYMMDDNN)
                              604800        ; Refresh (1 week)
                              86400         ; Retry (1 day)
                              2419200       ; Expire (4 weeks)
                              604800 )      ; Negative Cache TTL (1 week)
;
; Name Server
@       IN      NS      dns-server.example.local.

; A Records (IP Address)
@                       IN      A       192.168.1.10
dns-server              IN      A       192.168.1.10
www                     IN      A       192.168.1.20
mail                    IN      A       192.168.1.30
ftp                     IN      A       192.168.1.40
server1                 IN      A       192.168.1.50

; CNAME Records (Alias)
web                     IN      CNAME   www
webmail                 IN      CNAME   mail

; MX Record (Mail Server)
@                       IN      MX  10  mail.example.local.
```

**Sesuaikan:**
- Serial number: gunakan format `YYYYMMDDNN` (contoh: 2024102901)
- IP address sesuai kebutuhan
- Tambah/kurangi record sesuai kebutuhan

### 4.5 Buat File Reverse Zone

```bash
nano /etc/bind/zones/db.192.168.1
```

Isi dengan:
```
;
; BIND reverse data file for 192.168.1.x
;
$TTL    604800
@       IN      SOA     dns-server.example.local. admin.example.local. (
                              2024102901    ; Serial
                              604800        ; Refresh
                              86400         ; Retry
                              2419200       ; Expire
                              604800 )      ; Negative Cache TTL
;
; Name Server
@       IN      NS      dns-server.example.local.

; PTR Records (Reverse DNS)
10      IN      PTR     dns-server.example.local.
20      IN      PTR     www.example.local.
30      IN      PTR     mail.example.local.
40      IN      PTR     ftp.example.local.
50      IN      PTR     server1.example.local.
```

**Catatan:** Angka PTR record (10, 20, 30) adalah oktet terakhir dari IP address

### 4.6 Set Permission

```bash
# Set ownership ke user bind
chown -R bind:bind /etc/bind/zones/

# Set permission
chmod 755 /etc/bind/zones/
chmod 644 /etc/bind/zones/db.*
```

---

## Bagian 5: Validasi Konfigurasi

### 5.1 Cek Syntax Konfigurasi

```bash
# Cek konfigurasi utama
named-checkconf
```

Jika tidak ada output, berarti konfigurasi benar ✓

### 5.2 Cek Zone File

```bash
# Cek forward zone
named-checkzone example.local /etc/bind/zones/db.example.local

# Cek reverse zone
named-checkzone 1.168.192.in-addr.arpa /etc/bind/zones/db.192.168.1
```

Output yang benar:
```
zone example.local/IN: loaded serial 2024102901
OK
```

---

## Bagian 6: Menjalankan DNS Server

### 6.1 Restart BIND9

```bash
systemctl restart bind9
```

### 6.2 Cek Status

```bash
systemctl status bind9
```

Output yang benar: **active (running)** dengan warna hijau

### 6.3 Enable Auto Start

```bash
systemctl enable bind9
```

### 6.4 Cek Port DNS

```bash
# Cek port 53 (DNS)
ss -tulpn | grep :53
```

atau

```bash
netstat -tulpn | grep :53
```

Seharusnya muncul port 53 dengan status LISTEN

---

## Bagian 7: Konfigurasi DNS Client (Server)

Edit file resolv.conf:
```bash
nano /etc/resolv.conf
```

Ubah menjadi:
```
nameserver 127.0.0.1
search example.local
```

Untuk membuat persistent (tidak hilang saat reboot):
```bash
# Install resolvconf
apt install -y resolvconf

# Enable service
systemctl enable resolvconf.service
systemctl start resolvconf.service

# Edit konfigurasi
nano /etc/resolvconf/resolv.conf.d/head
```

Tambahkan:
```
nameserver 127.0.0.1
search example.local
```

Update resolv.conf:
```bash
resolvconf -u
```

---

## Bagian 8: Testing DNS Server

### 8.1 Test dengan nslookup

```bash
# Test forward lookup
nslookup dns-server.example.local

# Test domain lain
nslookup www.example.local
nslookup mail.example.local

# Test reverse lookup
nslookup 192.168.1.10
nslookup 192.168.1.20

# Test external domain (untuk cek forwarder)
nslookup google.com
```

### 8.2 Test dengan dig

```bash
# Forward lookup
dig dns-server.example.local

# Reverse lookup
dig -x 192.168.1.10

# Query detail
dig www.example.local +short

# Test ke external domain
dig @localhost google.com
```

### 8.3 Test dengan host

```bash
# Forward lookup
host dns-server.example.local

# Reverse lookup
host 192.168.1.10
```

### 8.4 Output yang Benar

Contoh output nslookup yang benar:
```
Server:         127.0.0.1
Address:        127.0.0.1#53

Name:   dns-server.example.local
Address: 192.168.1.10
```

---

## Bagian 9: Test dari Komputer Client

### 9.1 Setting DNS di Client Windows

1. Buka **Control Panel** → **Network and Sharing Center**
2. Klik **Change adapter settings**
3. Klik kanan pada adapter → **Properties**
4. Pilih **Internet Protocol Version 4 (TCP/IPv4)** → **Properties**
5. Pilih **Use the following DNS server addresses:**
   - Preferred DNS: `192.168.1.10` (IP DNS server Anda)
   - Alternate DNS: `8.8.8.8` (opsional)
6. Klik **OK**

Test di CMD:
```cmd
nslookup dns-server.example.local
nslookup www.example.local
ping dns-server.example.local
```

### 9.2 Setting DNS di Client Linux

Edit netplan (Ubuntu/Debian modern):
```bash
sudo nano /etc/netplan/01-netcfg.yaml
```

Tambahkan:
```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: yes
      nameservers:
        addresses: [192.168.1.10, 8.8.8.8]
        search: [example.local]
```

Apply:
```bash
sudo netplan apply
```

Test:
```bash
nslookup www.example.local
dig dns-server.example.local
```

---

## Bagian 10: Konfigurasi Firewall

### 10.1 Menggunakan UFW

```bash
# Install UFW
apt install -y ufw

# Allow DNS (port 53)
ufw allow 53/tcp
ufw allow 53/udp

# Allow SSH (agar tidak terputus)
ufw allow 22/tcp

# Enable firewall
ufw enable

# Cek status
ufw status
```

### 10.2 Menggunakan iptables

```bash
# Allow DNS
iptables -A INPUT -p tcp --dport 53 -j ACCEPT
iptables -A INPUT -p udp --dport 53 -j ACCEPT

# Save rules
apt install -y iptables-persistent
netfilter-persistent save
```

---

## Bagian 11: Monitoring dan Log

### 11.1 Lihat Log Real-time

```bash
# Monitor log BIND9
tail -f /var/log/syslog | grep named
```

### 11.2 Lihat Query Log (Opsional)

Edit `/etc/bind/named.conf.local`, tambahkan:
```
logging {
    channel query_log {
        file "/var/log/named/query.log" versions 3 size 5m;
        severity info;
        print-time yes;
        print-category yes;
    };
    category queries { query_log; };
};
```

Buat direktori log:
```bash
mkdir -p /var/log/named
chown bind:bind /var/log/named
```

Restart BIND9:
```bash
systemctl restart bind9
```

Lihat query log:
```bash
tail -f /var/log/named/query.log
```

---

## Bagian 12: Troubleshooting

### Masalah 1: DNS Server Tidak Merespon

```bash
# Cek status service
systemctl status bind9

# Cek error di log
grep -i error /var/log/syslog | grep named

# Restart service
systemctl restart bind9
```

### Masalah 2: Port 53 Sudah Digunakan

```bash
# Cek siapa yang pakai port 53
lsof -i :53
ss -tulpn | grep :53

# Kill process jika ada conflict
kill -9 [PID]
```

### Masalah 3: Zone Tidak Ter-load

```bash
# Cek syntax zone file
named-checkzone example.local /etc/bind/zones/db.example.local

# Cek permission
ls -la /etc/bind/zones/

# Fix permission
chown bind:bind /etc/bind/zones/db.*
chmod 644 /etc/bind/zones/db.*
```

### Masalah 4: Query Refused

Cek konfigurasi `allow-query` di `/etc/bind/named.conf.options`
```bash
allow-query { localhost; 192.168.1.0/24; };
```

Pastikan subnet sudah benar!

---

## Bagian 13: Menambah Record DNS Baru

### 13.1 Tambah A Record

Edit forward zone:
```bash
nano /etc/bind/zones/db.example.local
```

Tambahkan record baru:
```
server2    IN    A    192.168.1.60
```

**PENTING:** Increment serial number:
```
                              2024102902    ; Serial (naik 1)
```

### 13.2 Tambah PTR Record (Reverse)

Edit reverse zone:
```bash
nano /etc/bind/zones/db.192.168.1
```

Tambahkan:
```
60    IN    PTR    server2.example.local.
```

Increment serial number juga!

### 13.3 Reload Zone

```bash
# Reload tanpa restart
systemctl reload bind9

# Atau restart
systemctl restart bind9

# Test record baru
nslookup server2.example.local
```

---

## Bagian 14: Backup dan Restore

### 14.1 Backup Konfigurasi

```bash
# Backup semua konfigurasi
tar -czf bind-backup-$(date +%Y%m%d).tar.gz /etc/bind/

# Atau backup manual
cp -r /etc/bind/ /root/bind-backup-$(date +%Y%m%d)/
```

### 14.2 Restore

```bash
# Extract backup
tar -xzf bind-backup-YYYYMMDD.tar.gz -C /

# Restart BIND9
systemctl restart bind9
```

---

## Ringkasan Perintah Penting

```bash
# Restart DNS
systemctl restart bind9

# Reload tanpa restart
systemctl reload bind9

# Cek status
systemctl status bind9

# Cek konfigurasi
named-checkconf

# Cek zone
named-checkzone example.local /etc/bind/zones/db.example.local

# Test DNS
nslookup domain.example.local
dig domain.example.local
host domain.example.local

# Lihat log
tail -f /var/log/syslog | grep named

# Cek port
ss -tulpn | grep :53
```

---

## Checklist Konfigurasi

- [ ] IP Static sudah dikonfigurasi
- [ ] Hostname sudah diset
- [ ] BIND9 sudah terinstall
- [ ] File options sudah dikonfigurasi
- [ ] Zone forward sudah dibuat
- [ ] Zone reverse sudah dibuat
- [ ] Permission file sudah benar
- [ ] Konfigurasi sudah dicek dengan named-checkconf
- [ ] Zone file sudah dicek dengan named-checkzone
- [ ] BIND9 sudah running
- [ ] Port 53 sudah listening
- [ ] Test nslookup berhasil
- [ ] Test dari client berhasil
- [ ] Firewall sudah dikonfigurasi

---

## Selesai!

DNS Server Debian 12 Anda sudah siap digunakan!

**Tips:**
- Selalu increment serial number saat edit zone file
- Backup konfigurasi secara berkala
- Monitor log untuk deteksi masalah
- Update sistem secara rutin: `apt update && apt upgrade`
