# 🏢 Enterprise Network Infrastructure Simulation (GNS3)

Dokumentasi rancangan dan implementasi jaringan korporat (*Enterprise Network*) berskala kompleks menggunakan simulator **GNS3**. Proyek ini mendemonstrasikan kemampuan dalam merancang arsitektur jaringan multi-VLAN, otomatisasi layanan DHCP, serta penerapan kebijakan keamanan berlapis (*Security Policies*).

---

## 🛠️ Ringkasan Arsitektur & Perangkat
* **Simulator:** GNS3 (v2.2+)
* **Router:** Cisco c3745 (`R1`) — Berfungsi sebagai *Default Gateway* & *DHCP Server*.
* **Switch:** Built-in Ethernet Switch (`Switch1`) — Mengatur enkapsulasi Trunk (`dot1q`) dan Port Access VLAN.
* **End Devices:** Virtual PC Simulator (VPCS) untuk simulasi host per departemen.

---

## 📊 Alokasi VLAN & Subnetting (/25)
Jaringan dibagi menjadi 5 departemen operasional menggunakan subnetting prefiks **/25** (`255.255.255.128`), yang masing-masing mendukung kapasitas hingga 126 host *usable* untuk mengantisipasi ekspansi perusahaan:

| Departemen | VLAN ID | Subnet / Mask | Gateway (Router) | Status DHCP Pool |
| :--- | :---: | :--- | :--- | :--- |
| **Manajemen** | VLAN 10 | `192.168.10.0/25` | `192.168.10.1` | Active / Tested |
| **HR-Admin** | VLAN 20 | `192.168.20.0/25` | `192.168.20.1` | Active / Tested |
| **Finance** | VLAN 30 | `192.168.30.0/25` | `192.168.30.1` | Active / Protected |
| **IT-Server** | VLAN 40 | `192.168.40.0/25` | `192.168.40.1` | Active / Tested |
| **Operational** | VLAN 50 | `192.168.50.0/25` | `192.168.50.1` | Active / Restricted |

---

## ⚙️ Skrip Konfigurasi Utama

### 1. Konfigurasi Router R1 (Router-on-a-Stick & DHCP Server)
<img width="1816" height="813" alt="topologi-gns3" src="https://github.com/user-attachments/assets/93c4ebb6-1720-438b-b860-7f7b2cb06d8f" />

```text
enable
configure terminal
hostname R1

! --- Pengecualian IP untuk Gateway & Static ---
ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp excluded-address 192.168.20.1 192.168.20.10
ip dhcp excluded-address 192.168.30.1 192.168.30.10
ip dhcp excluded-address 192.168.40.1 192.168.40.10
ip dhcp excluded-address 192.168.50.1 192.168.50.10

! --- Contoh DHCP Pool: Manajemen ---
ip dhcp pool Manajemen
 network 192.168.10.0 255.255.255.128
 default-router 192.168.10.1
 dns-server 8.8.8.8

! --- Konfigurasi Interface Fisik & Sub-Interface (Router-on-a-Stick) ---
interface FastEthernet0/0
 no shutdown

interface FastEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.128

interface FastEthernet0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.128

interface FastEthernet0/0.50
 encapsulation dot1Q 50
 ip address 192.168.50.1 255.255.255.128
```

### 2. Keamanan Jaringan (Extended ACL)
Menerapkan *Access Control List* pada router untuk mengamankan data sensitif departemen **Finance (VLAN 30)** dari akses ilegal departemen **Operational (VLAN 50)**:
<img width="638" height="364" alt="dhcp-routing-test" src="https://github.com/user-attachments/assets/f1ce2158-9b56-4f5f-a0ef-525875599b07" />
<img width="689" height="383" alt="acl-test-result" src="https://github.com/user-attachments/assets/6bd2d6bb-8562-42f0-9966-50f4bf250cc0" />


```text
ip access-list extended KEAMANAN-KANTOR
 deny ip 192.168.50.0 0.0.0.127 192.168.30.0 0.0.0.127
 permit ip any any

interface FastEthernet0/0.50
 ip access-group KEAMANAN-KANTOR in
 ```<img width="1816" height="813" alt="topologi-gns3" src="https://github.com/user-attachments/assets/6d302e33-d903-47de-b672-72d055a5c5ff" />
<img width="638" height="364" alt="dhcp-routing-test" src="https://github.com/user-attachments/assets/e3c887e6-d883-45b1-bf12-6a823c752dd0" />
