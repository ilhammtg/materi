````md
# Tutorial Konfigurasi FTP Server di Debian 13 (VirtualBox)

## Pendahuluan
FTP (File Transfer Protocol) digunakan untuk transfer file antar komputer dalam jaringan. Pada tutorial ini kita akan menggunakan **vsftpd (Very Secure FTP Daemon)**.

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

## Install vsftpd

```bash
sudo apt install vsftpd -y
```

Cek status:

```bash
sudo systemctl status vsftpd
```

---

## Konfigurasi vsftpd

Backup config default:

```bash
sudo cp /etc/vsftpd.conf /etc/vsftpd.conf.bak
```

Edit file konfigurasi:

```bash
sudo nano /etc/vsftpd.conf
```

Ubah / tambahkan konfigurasi berikut:

```conf
listen=YES
listen_ipv6=NO

anonymous_enable=NO
local_enable=YES
write_enable=YES

chroot_local_user=YES
allow_writeable_chroot=YES

pasv_enable=YES
pasv_min_port=10000
pasv_max_port=10100

user_sub_token=$USER
local_root=/home/$USER/ftp

```

Simpan: `CTRL + O`, Enter
Keluar: `CTRL + X`

---

## Membuat User FTP

Buat user baru:

```bash
sudo adduser ftpuser
```

---

## Setup Direktori FTP

```bash
sudo mkdir -p /home/ftpuser/ftp/upload
sudo chown nobody:nogroup /home/ftpuser/ftp
sudo chmod a-w /home/ftpuser/ftp

sudo chown ftpuser:ftpuser /home/ftpuser/ftp/upload
```

---

## Restart Service

```bash
sudo systemctl restart vsftpd
```

---

## Konfigurasi Firewall

Jika menggunakan UFW:

```bash
sudo ufw allow 21/tcp
sudo ufw allow 10000:10100/tcp
sudo ufw reload
```

---

## Testing FTP

### Dari Client (CMD / Terminal):

```bash
ftp <IP_SERVER>
```

Login:

```
Username: ftpuser
Password: (password yang dibuat)
```

---

## Cek IP Address Server

```bash
ip a
```

Gunakan IP tersebut untuk koneksi dari host/PC lain.

---

## Catatan Penting

* Gunakan mode **Bridged Adapter** di VirtualBox agar bisa diakses dari jaringan lokal
* FTP tidak aman (plain text), untuk produksi gunakan **SFTP (SSH)**

---

## Selesai

FTP Server sudah berhasil dikonfigurasi dan siap digunakan

```
