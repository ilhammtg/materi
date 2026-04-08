````md id="wsb001"
# Tutorial Web Server Dasar di Debian 13 (VirtualBox)

## Pendahuluan
Web server adalah layanan yang digunakan untuk menampilkan halaman website melalui browser.  
Pada tutorial ini kita hanya menggunakan:
- Apache2 (Web Server)
- HTML sederhana

---

## Persiapan
Pastikan:
- Debian 13 sudah terinstall di VirtualBox
- Koneksi internet aktif
- Login sebagai root atau user dengan sudo

---

## Update Sistem
```bash
sudo apt update && sudo apt upgrade -y
````

---

## Install Apache

```bash
sudo apt install apache2 -y
```

---

## Cek Status Apache

```bash
sudo systemctl status apache2
```

Jika muncul status **active (running)** berarti Apache sudah berjalan.

---

## Cek IP Address Server

```bash
ip a
```

Catat IP, contoh:

```text
192.168.1.10
```

---

## Testing Web Server

Buka browser di komputer host, lalu akses:

```text
http://IP_SERVER
```

Jika muncul halaman **Apache2 Default Page**, berarti web server berhasil.

---

## Struktur Folder Web

Folder utama web:

```text
/var/www/html/
```

---

## Membuat Website Sederhana

Edit file index:

```bash
sudo nano /var/www/html/index.html
```

Isi dengan:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Web Server Debian</title>
</head>
<body>
    <h1>Web Server Berhasil!</h1>
    <p>Ini adalah halaman web pertama saya.</p>
</body>
</html>
```

Simpan:

* CTRL + O → Enter
* CTRL + X

---

## Restart Apache (Opsional)

```bash
sudo systemctl restart apache2
```

---

## Cek Hasil

Buka kembali di browser:

```text
http://IP_SERVER
```

Jika tampil halaman yang dibuat, berarti berhasil

---

## (Opsional) Firewall

Jika menggunakan UFW:

```bash
sudo ufw allow 80/tcp
sudo ufw reload
```

---

## Catatan Penting

* Gunakan **Bridged Adapter** di VirtualBox agar bisa diakses dari komputer lain
* Pastikan Apache berjalan saat boot:

```bash
sudo systemctl enable apache2
```

---

## Kesimpulan

* Apache berhasil diinstall
* Web server berjalan
* Halaman HTML bisa diakses dari browser

Web server dasar sudah siap digunakan

```
