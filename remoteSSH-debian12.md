# Tutorial Konfigurasi Remote SSH Debian 12 di VirtualBox

## Pendahuluan
Tutorial ini akan memandu Anda untuk mengkonfigurasi SSH Server pada Debian 12 yang berjalan di VirtualBox, sehingga dapat diakses dari Windows host melalui Command Prompt atau Terminal.

## Prasyarat
- VirtualBox sudah terinstall
- Debian 12 sudah terinstall di VirtualBox
- Koneksi internet aktif

---

## Bagian 1: Konfigurasi Network VirtualBox

### 1.1 Setting Network Adapter
1. Matikan VM Debian 12 terlebih dahulu
2. Buka VirtualBox Manager, pilih VM Debian 12
3. Klik **Settings** → **Network**
4. Pada **Adapter 1**, pastikan setting sebagai berikut:
   - **Attached to**: NAT
5. Klik **Advanced** → **Port Forwarding**
6. Tambahkan rule baru dengan klik tombol **+**:
   - **Name**: SSH
   - **Protocol**: TCP
   - **Host IP**: 127.0.0.1
   - **Host Port**: 2222 (atau port lain yang belum digunakan)
   - **Guest IP**: (kosongkan)
   - **Guest Port**: 22
7. Klik **OK** untuk menyimpan

### Alternatif: Menggunakan Bridged Adapter
Jika ingin menggunakan IP lokal langsung:
1. Pada **Adapter 1**, ubah **Attached to**: Bridged Adapter
2. **Name**: Pilih adapter jaringan aktif dari host (WiFi atau Ethernet)
3. Tidak perlu Port Forwarding dengan metode ini

---

## Bagian 2: Instalasi dan Konfigurasi SSH Server di Debian 12

### 2.1 Login ke Debian 12
Nyalakan VM Debian 12 dan login dengan user Anda.

### 2.2 Update Sistem
```bash
sudo apt update
sudo apt upgrade -y
```

### 2.3 Install OpenSSH Server
```bash
sudo apt install openssh-server -y
```

### 2.4 Cek Status SSH Service
```bash
sudo systemctl status ssh
```

Pastikan status menunjukkan **active (running)**. Jika belum aktif, jalankan:
```bash
sudo systemctl start ssh
sudo systemctl enable ssh
```

### 2.5 Konfigurasi SSH (Opsional)
Edit file konfigurasi SSH jika diperlukan:
```bash
sudo nano /etc/ssh/sshd_config
```

Setting yang umum diubah:
- `Port 22` - Port SSH (default 22)
- `PermitRootLogin no` - Melarang login sebagai root
- `PasswordAuthentication yes` - Mengizinkan login dengan password

Setelah selesai edit, tekan `Ctrl+X`, lalu `Y`, dan `Enter` untuk menyimpan.

Restart service SSH:
```bash
sudo systemctl restart ssh
```

### 2.6 Cek IP Address Debian
**Jika menggunakan NAT + Port Forwarding**: Tidak perlu cek IP, akan menggunakan localhost.

**Jika menggunakan Bridged Adapter**: Catat IP address dengan perintah:
```bash
ip addr show
```
atau
```bash
hostname -I
```

Cari IP yang dimulai dengan 192.168.x.x atau 10.x.x.x

### 2.7 Konfigurasi Firewall (Jika Aktif)
Jika menggunakan UFW (Uncomplicated Firewall), izinkan SSH:
```bash
sudo ufw allow ssh
sudo ufw enable
sudo ufw status
```

---

## Bagian 3: Testing Koneksi SSH dari Windows

### 3.1 Menggunakan Command Prompt atau PowerShell

#### Jika Menggunakan NAT + Port Forwarding:
Buka Command Prompt atau PowerShell di Windows, lalu jalankan:
```cmd
ssh username@localhost -p 2222
```

Ganti `username` dengan username Debian Anda.

#### Jika Menggunakan Bridged Adapter:
```cmd
ssh username@IP_ADDRESS_DEBIAN
```

Ganti `username` dan `IP_ADDRESS_DEBIAN` dengan data yang sesuai.

### 3.2 Koneksi Pertama Kali
Saat pertama kali connect, akan muncul peringatan:
```
The authenticity of host can't be established.
Are you sure you want to continue connecting (yes/no)?
```

Ketik `yes` dan tekan Enter.

Masukkan password user Debian Anda.

### 3.3 Verifikasi Koneksi
Jika berhasil, Anda akan melihat prompt Debian di terminal Windows:
```
username@debian:~$
```

Coba jalankan beberapa perintah untuk memastikan:
```bash
whoami
pwd
uname -a
```

---

## Troubleshooting

### SSH Connection Refused
- Pastikan SSH service berjalan di Debian: `sudo systemctl status ssh`
- Cek firewall tidak memblokir port 22: `sudo ufw status`
- Pastikan port forwarding sudah dikonfigurasi dengan benar (untuk NAT)

### Tidak Bisa Ping dari Windows ke Debian
- Jika menggunakan NAT, ini normal. NAT tidak support ping ICMP dari host ke guest
- Gunakan Bridged Adapter jika perlu ping

### Permission Denied
- Pastikan username dan password benar
- Cek konfigurasi `/etc/ssh/sshd_config`, pastikan `PasswordAuthentication yes`

### Port Sudah Digunakan (untuk Port Forwarding)
- Ganti Host Port dari 2222 ke port lain yang tersedia (misalnya 2223, 3333, dll)
- Lalu gunakan port tersebut saat SSH: `ssh username@localhost -p PORT_BARU`

---

## Tips Tambahan

### Menggunakan SSH Key (Lebih Aman)
Generate SSH key di Windows:
```cmd
ssh-keygen -t rsa -b 4096
```

Copy public key ke Debian:
```cmd
type %USERPROFILE%\.ssh\id_rsa.pub | ssh username@localhost -p 2222 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

Setelah itu, Anda bisa login tanpa password.

### Mematikan Password Authentication (Setelah Setup SSH Key)
Edit `/etc/ssh/sshd_config`:
```bash
PasswordAuthentication no
```

Restart SSH:
```bash
sudo systemctl restart ssh
```

---

## Kesimpulan
Anda sekarang sudah berhasil mengkonfigurasi SSH Server pada Debian 12 di VirtualBox dan dapat mengaksesnya dari Windows melalui Command Prompt atau PowerShell. Untuk keamanan lebih baik, disarankan menggunakan SSH key authentication dan menonaktifkan password authentication.

**Selamat mencoba!**
