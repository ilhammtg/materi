# 📡 Instalasi & Konfigurasi DHCP Server  
## Debian 12 (CLI)

Dokumentasi ini menjelaskan langkah-langkah **instalasi dan konfigurasi DHCP Server pada Debian 12 menggunakan Command Line Interface (CLI)**.

---

## 1. Pengertian DHCP Server

**DHCP (Dynamic Host Configuration Protocol)** adalah layanan jaringan yang berfungsi untuk **memberikan alamat IP secara otomatis** kepada client dalam satu jaringan, termasuk:

* IP Address
* Subnet Mask
* Default Gateway
* DNS Server

Tanpa DHCP, administrator harus mengatur IP client **secara manual**, yang tidak efisien untuk jaringan besar.

---

## 2. Topologi Jaringan (Ilustrasi)

![Image](https://svg.template.creately.com/iqj71w2z3)

![Image](https://linuxhint.com/wp-content/uploads/2019/04/1-15.png)

![Image](https://docs.oracle.com/cd/E23823_01/html/816-4554/figures/dhcp-diag.png)

**Keterangan:**

* Debian 12 berperan sebagai **DHCP Server**
* Client menerima IP otomatis
* Semua berada dalam satu jaringan LAN

---

## 3. Persiapan Sistem

Sebelum instalasi, pastikan:

* Debian 12 sudah terpasang
* Login sebagai **root** atau user dengan akses sudo
* Network interface sudah dikenali

### Cek interface jaringan

```bash
ip a
```

Contoh interface:

```
enp0s3
```

---

## 4. Instalasi DHCP Server

Debian 12 menggunakan paket **isc-dhcp-server**.

### Update repository

```bash
apt update
```

### Install DHCP Server

```bash
apt install isc-dhcp-server -y
```

---

## 5. Konfigurasi Interface DHCP

Edit file berikut:

```bash
nano /etc/default/isc-dhcp-server
```

Ubah bagian:

```ini
INTERFACESv4="enp0s3"
```

**Catatan:**
Sesuaikan `enp0s3` dengan nama interface jaringan Anda.

---

## 6. Konfigurasi DHCP Server

Edit file konfigurasi utama:

```bash
nano /etc/dhcp/dhcpd.conf
```

### Contoh konfigurasi sederhana

```conf
option domain-name "localnet";
option domain-name-servers 8.8.8.8, 8.8.4.4;

default-lease-time 600;
max-lease-time 7200;

authoritative;

subnet 192.168.10.0 netmask 255.255.255.0 {
    range 192.168.10.10 192.168.10.50;
    option routers 192.168.10.1;
    option subnet-mask 255.255.255.0;
    option broadcast-address 192.168.10.255;
}
```

### Penjelasan Singkat

| Konfigurasi                | Fungsi                  |
| -------------------------- | ----------------------- |
| range                      | Rentang IP untuk client |
| option routers             | Gateway                 |
| option domain-name-servers | DNS                     |
| authoritative              | Server utama DHCP       |

---

## 7. Restart dan Aktifkan DHCP Server

```bash
systemctl restart isc-dhcp-server
systemctl enable isc-dhcp-server
```

### Cek status layanan

```bash
systemctl status isc-dhcp-server
```

Jika berhasil, status akan **active (running)**.

---

## 8. Pengujian DHCP

### Pada Client Linux

```bash
dhclient
```

### Cek IP yang diterima

```bash
ip a
```

### Cek daftar lease DHCP

```bash
cat /var/lib/dhcp/dhcpd.leases
```

---

## 9. Troubleshooting Umum

### DHCP tidak berjalan

* Pastikan interface benar
* Pastikan subnet sesuai dengan IP server
* Cek error:

```bash
journalctl -xe
```

### Client tidak dapat IP

* Pastikan client satu jaringan
* Pastikan tidak ada DHCP lain dalam jaringan
