# 🧠 Cisco Packet Tracer VLAN + DHCP + Static Routing Setup
**Topologi: 3 Router + 4 Switch + 8 PC**

---

## 🗺️ Topologi Ringkas

```
[PC0][PC1]──SW-LEFT──R-LEFT──R-MID──R-RIGHT──SW-RIGHT-V10──[PC4][PC5]
                                            └──SW-RIGHT-V20──[PC6][PC7]
                    │
                    └──SW-MID──(VLAN10 & VLAN20)
```

---

## ⚙️ 1. IP Address Planning

| Device / Segment | VLAN | Network/Subnet | Gateway (Router) | Notes |
|------------------|-------|----------------|------------------|--------|
| SW-LEFT (VLAN10) | 10 | 192.168.10.0/24 | 192.168.10.254 | PC0, PC1 |
| R-LEFT ↔ R-MID | - | 10.10.10.0/24 | - | Inter-router link |
| SW-MID VLAN10 | 10 | 192.168.10.0/24 | 192.168.10.1 | PC2 |
| SW-MID VLAN20 | 20 | 192.168.20.0/24 | 192.168.20.1 | PC3 |
| R-MID ↔ R-RIGHT | - | 11.11.11.0/24 | - | Inter-router link |
| SW-RIGHT-V10 | 10 | 192.168.10.0/24 | 192.168.10.2 | PC4, PC5 |
| SW-RIGHT-V20 | 20 | 192.168.20.0/24 | 192.168.20.2 | PC6, PC7 |

> **Catatan:** Karena VLAN10 dan VLAN20 digunakan di banyak site dengan subnet sama, semua router harus menghubungkan antar-site via **static routing Layer-3**, bukan Layer-2 trunk lintas-site.

---

## ⚙️ 2. SWITCH PALING KIRI (SW-LEFT)

```bash
enable
conf t
vlan 10
 name VLAN10_LEFT
!
int range fa0/1 - 2
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
!
int fa0/3
 switchport mode trunk
 switchport trunk allowed vlan 10
end
wr
```

---

## ⚙️ 3. ROUTER PALING KIRI (R-LEFT)

```bash
enable
conf t
# Interface ke SW-LEFT (router-on-a-stick)
int fa0/0
 no ip address
 no shut
!
int fa0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.254 255.255.255.0
 no shut
!
# Link ke R-MID
int gig6/0
 ip address 10.10.10.1 255.255.255.0
 no shut
!
# DHCP Server VLAN10
ip dhcp excluded-address 192.168.10.1 192.168.10.20
ip dhcp pool VLAN10
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.254
 dns-server 8.8.8.8
!
# Routing
ip route 192.168.20.0 255.255.255.0 10.10.10.2
ip route 11.11.11.0 255.255.255.0 10.10.10.2
end
wr
```

---

## ⚙️ 4. SWITCH TENGAH (SW-MID)

```bash
enable
conf t
vlan 10
 name VLAN10_MID
vlan 20
 name VLAN20_MID
!
int fa0/10
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
!
int fa0/11
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
!
int fa0/4
 switchport mode trunk
 switchport trunk allowed vlan 10,20
end
wr
```

---

## ⚙️ 5. ROUTER TENGAH (R-MID)

```bash
enable
conf t
# ke switch tengah
int fa0/4
 no ip address
 no shut
!
int fa0/4.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
!
int fa0/4.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
!
# ke R-LEFT
int gig6/0
 ip address 10.10.10.2 255.255.255.0
 no shut
!
# ke R-RIGHT
int gig7/0
 ip address 11.11.11.1 255.255.255.0
 no shut
!
# DHCP VLAN10 dan VLAN20
ip dhcp excluded-address 192.168.10.1 192.168.10.20
ip dhcp excluded-address 192.168.20.1 192.168.20.20
ip dhcp pool VLAN10
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8
ip dhcp pool VLAN20
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 8.8.8.8
!
# Routing
ip route 192.168.10.0 255.255.255.0 10.10.10.1
ip route 192.168.20.0 255.255.255.0 11.11.11.2
end
wr
```

---

## ⚙️ 6. ROUTER PALING KANAN (R-RIGHT)

```bash
enable
conf t
# VLAN10 (ke SW kanan pertama)
int fa0/5
 ip address 192.168.10.2 255.255.255.0
 no shut
# VLAN20 (ke SW kanan kedua)
int fa1/0
 ip address 192.168.20.2 255.255.255.0
 no shut
# ke R-MID
int gig7/0
 ip address 11.11.11.2 255.255.255.0
 no shut
!
# DHCP Server VLAN kanan
ip dhcp excluded-address 192.168.10.1 192.168.10.20
ip dhcp excluded-address 192.168.20.1 192.168.20.20
ip dhcp pool VLAN10
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.2
 dns-server 8.8.8.8
ip dhcp pool VLAN20
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.2
 dns-server 8.8.8.8
!
# Routing
ip route 192.168.10.0 255.255.255.0 11.11.11.1
ip route 192.168.20.0 255.255.255.0 11.11.11.1
ip route 10.10.10.0 255.255.255.0 11.11.11.1
end
wr
```

---

## ⚙️ 7. SWITCH KANAN VLAN10 (SW-RIGHT-V10)

```bash
enable
conf t
vlan 10
 name VLAN10_RIGHT
!
int range fa0/12 - 13
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
!
int fa0/5
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
end
wr
```

---

## ⚙️ 8. SWITCH KANAN VLAN20 (SW-RIGHT-V20)

```bash
enable
conf t
vlan 20
 name VLAN20_RIGHT
!
int range fa0/20 - 21
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
!
int fa0/6
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
end
wr
```

---

## ✅ 9. PENGUJIAN

### Cek IP Interface Router
```
show ip interface brief
```

### Cek DHCP Binding
```
show ip dhcp binding
```

### Cek Static Route
```
show ip route
```

### Uji Koneksi
1. Ping dari **PC0 → Gateway 192.168.10.254**
2. Ping dari **PC0 → PC2 (VLAN10 di router tengah)**
3. Ping dari **PC6 → PC7 (VLAN20 kanan)**
4. Ping dari **PC0 → PC7** (melewati semua router)

---

## ⚠️ Tips Penting
- Interface fisik (`fa0/0`) di router-on-a-stick **tidak boleh diberi IP** langsung.  
- Pastikan **switch ke router menggunakan trunk**.  
- Cek selalu `show vlan brief` dan `show interfaces trunk` di switch.  
- DHCP server hanya aktif jika **service dhcp** ON:
  ```bash
  service dhcp
  ```

---

**Selesai ✅**
Jaringan VLAN 10 & 20 sudah saling terhubung antar-site, dengan DHCP otomatis dan routing antar-router.
