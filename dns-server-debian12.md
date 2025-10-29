# Tutorial Instalasi & Konfigurasi DNS Server di Debian (VirtualBox)

## Pendahuluan

DNS (Domain Name System) berfungsi untuk menerjemahkan nama domain (seperti `example.com`) menjadi alamat IP (seperti `192.168.1.10`).

Pada tutorial ini, kamu akan belajar:

1. Menetapkan IP statis di Debian (di VirtualBox)
2. Menginstal paket DNS Server (BIND9)
3. Mengonfigurasi DNS server utama (master)
4. Menambahkan zona langsung (forward) dan zona balik (reverse)
5. Menguji hasil konfigurasi

---

## 1. Konfigurasi IP Address Statis di Debian

### Langkah 1: Buka terminal Debian

Pastikan kamu sudah login sebagai `root` atau gunakan `sudo`.

### Langkah 2: Edit file konfigurasi network

```bash
sudo nano /etc/network/interfaces
```

### Langkah 3: Ubah konfigurasi menjadi seperti berikut (contoh):

```bash
# Interface utama
auto enp0s3
iface enp0s3 inet static
    address 192.168.10.1
    netmask 255.255.255.0
    network 192.168.10.0
```

> **Catatan:**
>
> * `enp0s3` bisa berbeda tergantung adapter VirtualBox kamu (cek dengan `ip a`).
> * IP `192.168.10.1` adalah IP statis untuk DNS server kamu.

### Langkah 4: Restart network service

```bash
sudo systemctl restart networking
```

### Langkah 5: Verifikasi IP

```bash
ip a
```

Pastikan IP sudah berubah sesuai konfigurasi.

---

## ⚙️ 2. Instalasi DNS Server (BIND9)

### Jalankan perintah berikut:

```bash
sudo apt update
sudo apt install bind9 bind9utils bind9-doc -y
```

Setelah selesai, cek status BIND9:

```bash
sudo systemctl status bind9
```

Jika berjalan dengan baik, tampilannya harus `active (running)`.

---

## 3. Konfigurasi Zona DNS

### Langkah 1: Edit file utama konfigurasi zona

```bash
sudo nano /etc/bind/named.conf.local
```

Tambahkan konfigurasi zona berikut:

```bash
zone "contoh.local" {
    type master;
    file "/etc/bind/db.contoh.local";
};

zone "56.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.192";
};
```

> **Penjelasan:**
>
> * `contoh.local` = nama domain lokal (bisa kamu ganti)
> * `db.contoh.local` = file zona forward
> * `db.192` = file zona reverse untuk jaringan `192.168.10.0/24`

---

## 4. Membuat File Zona Forward dan Reverse

### File Zona Forward

Salin template default dan ubah:

```bash
sudo cp /etc/bind/db.local /etc/bind/db.contoh.local
sudo nano /etc/bind/db.contoh.local
```

Ubah isinya seperti berikut:

```dns
$TTL    604800
@       IN      SOA     contoh.local. root.contoh.local. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns.contoh.local.
ns      IN      A       192.168.10.1
server  IN      A       192.168.10.1
client  IN      A       192.168.10.11
```

### File Zona Reverse

```bash
sudo cp /etc/bind/db.127 /etc/bind/db.192
sudo nano /etc/bind/db.192
```

Ubah isinya seperti berikut:

```dns
$TTL    604800
@       IN      SOA     contoh.local. root.contoh.local. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns.contoh.local.
10      IN      PTR     ns.contoh.local.
10      IN      PTR     server.contoh.local.
11      IN      PTR     client.contoh.local.
```

---

## 5. Uji Konfigurasi DNS

### Langkah 1: Cek kesalahan konfigurasi

```bash
sudo named-checkconf
sudo named-checkzone contoh.local /etc/bind/db.contoh.local
sudo named-checkzone 56.168.192.in-addr.arpa /etc/bind/db.192
```

Jika semua hasilnya **OK**, lanjut ke langkah berikut.

### Langkah 2: Restart layanan BIND9

```bash
sudo systemctl restart bind9
```

### Langkah 3: Tes dengan `dig` atau `nslookup`

```bash
dig server.contoh.local
nslookup client.contoh.local
```

Jika konfigurasi benar, hasilnya akan menampilkan IP yang sesuai (`192.168.10.1` atau `192.168.10.11`).

---

## 6. Tips Tambahan

* Jika DNS tidak bisa di-resolve, pastikan **adapter VirtualBox** dalam mode **Host-Only Adapter**.
* Tambahkan IP DNS (`192.168.10.1`) ke `/etc/resolv.conf` agar sistem menggunakan DNS lokal.

  ```bash
  sudo nano /etc/resolv.conf
  nameserver 192.168.10.1
  ```
* Gunakan perintah ini untuk melihat log jika ada error:

  ```bash
  sudo tail -f /var/log/syslog
  ```

---

## Penutup

Kamu telah berhasil membuat dan mengonfigurasi DNS server di Debian menggunakan BIND9! Sekarang sistem kamu bisa menerjemahkan nama domain lokal menjadi IP address dan sebaliknya.

> **Selanjutnya:** Kamu bisa menambahkan client Debian lain di VirtualBox dan mengarahkannya ke IP DNS ini untuk menguji resolusi domain dari komputer lain.
