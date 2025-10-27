# Tutorial Konfigurasi DNS Server pada Debian 12 (CLI) di VirtualBox

## Pendahuluan

DNS (Domain Name System) adalah layanan yang menerjemahkan nama domain menjadi alamat IP agar komputer dapat saling berkomunikasi. Dalam tutorial ini, kita akan mengkonfigurasi **DNS Server menggunakan BIND9** di **Debian 12** yang dijalankan melalui **VirtualBox**.

---

## Persiapan Awal

### 1. Konfigurasi VirtualBox

* Buat **Virtual Machine Debian 12** dengan spesifikasi minimal:

  * RAM: 1 GB atau lebih
  * Storage: 10 GB
  * Adapter jaringan: **Bridged Adapter** atau **Internal Network**

### 2. Cek Koneksi Internet

```bash
ping google.com
```

Pastikan sistem memiliki akses internet untuk instalasi paket.

### 3. Update Repository

```bash
sudo apt update && sudo apt upgrade -y
```

---

## Instalasi BIND9

Instal paket DNS Server utama:

```bash
sudo apt install bind9 bind9utils bind9-doc -y
```

Cek status layanan:

```bash
sudo systemctl status bind9
```

Pastikan status menunjukkan **active (running)**.

---

## Konfigurasi DNS Server

### 1. Edit File Konfigurasi Utama

File utama berada di `/etc/bind/named.conf.options`

```bash
sudo nano /etc/bind/named.conf.options
```

Ubah isinya seperti berikut:

```bash
options {
    directory "/var/cache/bind";

    recursion yes;
    allow-query { any; };

    forwarders {
        8.8.8.8;
        1.1.1.1;
    };

    dnssec-validation no;
    listen-on { any; };
    listen-on-v6 { any; };
};
```

Simpan dengan `Ctrl+O`, lalu keluar dengan `Ctrl+X`.

---

### 2. Tambah Zona Forward dan Reverse

Edit file konfigurasi zona:

```bash
sudo nano /etc/bind/named.conf.local
```

Tambahkan konfigurasi zona berikut:

```bash
zone "ilham.local" {
    type master;
    file "/etc/bind/db.ilham.local";
};

zone "1.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.192";
};
```

> **Catatan:**
> `ilham.local` adalah domain buatan. Ganti dengan nama domain Anda sendiri bila perlu.

---

### 3. Buat File Zona Forward

Salin template bawaan dan ubah:

```bash
sudo cp /etc/bind/db.local /etc/bind/db.ilham.local
sudo nano /etc/bind/db.ilham.local
```

Isi dengan contoh berikut:

```bash
$TTL    604800
@       IN      SOA     ns1.ilham.local. root.ilham.local. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns1.ilham.local.
ns1     IN      A       192.168.1.10
www     IN      A       192.168.1.10
```

---

### 4. Buat File Zona Reverse

```bash
sudo cp /etc/bind/db.127 /etc/bind/db.192
sudo nano /etc/bind/db.192
```

Isi dengan:

```bash
$TTL    604800
@       IN      SOA     ns1.ilham.local. root.ilham.local. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns1.ilham.local.
10      IN      PTR     ns1.ilham.local.
```

---

## Restart dan Uji DNS

### 1. Restart BIND9

```bash
sudo systemctl restart bind9
```

### 2. Cek Status

```bash
sudo systemctl status bind9
```

Pastikan status **active (running)**.

### 3. Uji Resolusi DNS

Gunakan perintah:

```bash
dig @localhost ilham.local
```

atau

```bash
nslookup ilham.local 127.0.0.1
```

Jika konfigurasi benar, maka IP address akan muncul sesuai zona yang dibuat.

---

## Tips Tambahan

* Jika ingin klien lain (VM atau PC) menggunakan DNS ini, atur **nameserver** di `/etc/resolv.conf`:

  ```bash
  nameserver 192.168.1.10
  ```
* Untuk melihat log error:

  ```bash
  sudo tail -f /var/log/syslog
  ```

---

## Penutup

Sekarang Anda telah berhasil mengkonfigurasi **DNS Server di Debian 12** menggunakan **BIND9** pada **VirtualBox**. DNS ini dapat digunakan untuk keperluan lokal seperti jaringan laboratorium, kampus, atau simulasi server internal.

> 🧑‍💻 **Disusun oleh:** Ilham
> 📅 **Versi:** 1.0 — Oktober 2025
