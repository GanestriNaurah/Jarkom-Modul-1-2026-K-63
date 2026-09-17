# Jarkom-Modul-1-2026-K-63

| Nama | NRP |
| :---: | :---: |
| Nadya Putri Agustin | 5027251013 |
| Ganestri Naurah Sawestri | 5027251014 |

Host (IP Controller): `10.4.89.250`

## 1. Membuat Topologi Terintegrasi
Untuk mempersiapkan pembangunan The Wired, Lain yang berperan sebagai Router membuat tiga Switch/Gateway: Switch 1 menuju dua Entitas yaitu Alice dan Mika, Switch 2 menuju Chisa, sedangkan Switch 3 menuju Knights dan Eiri. Kelima Entitas tersebut dikonfigurasi sebagai Client di GNS3. 

<img width="1561" height="655" alt="Image" src="https://github.com/user-attachments/assets/910682e2-401d-43cd-8226-bf9b74e1cb46" />

## 2.  Konfigurasi Router 

Karena menurut Lain pada saat itu The Wired masih terisolasi dari dunia luar, konfigurasikan router Lain agar dapat tersambung langsung ke jaringan internet publik melalui NAT/DHCP pada interface eth0.

Setelah router Lain terhubung ke internet, pastikan seluruh Entitas (Client) di bawah Switch 1, Switch 2, dan Switch 3 dapat saling terhubung dan berkomunikasi satu sama lain melalui konfigurasi routing.


#### Konfigurasi Jaringan pada Router
```
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static 
     address 10.95.1.1
     netmask 255.255.255.0

auto eth2
iface eth2 inet static
     address 10.95.2.1
     netmask 255.255.255.0

auto eth3
iface eth3 inet static
     address 10.95.3.1
     netmask 255.255.255.0
```

## 3.  Konfigurasi pada Switch 

#### Konfigurasi pada Switch 1

- Client Alice
```
auto eth0
iface eth0 inet static
     address 10.95.1.2
     netmask 255.255.255.0
     gateway 10.95.1.1
```
- Client Mika
```
auto eth0
iface eth0 inet static
     address 10.95.1.3
     netmask 255.255.255.0
     gateway 10.95.1.1
```

#### Konfigurasi pada Switch 2
- Client Chisa
```
auto eth0
iface eth0 inet static
     address 10.95.2.2
     netmask 255.255.255.0
     gateway 10.95.2.1
```

#### Konfigurasi pada Switch 3
- Client Knights
```
auto eth0
iface eth0 inet static
     address 10.95.3.2
     netmask 255.255.255.0
     gateway 10.95.3.1
```
- Client Eiri
```
auto eth0
iface eth0 inet static
     address 10.95.3.3
     netmask 255.255.255.0
     gateway 10.95.3.1
```

Pastikan semua salig terhubung menggunakan:
```
ping -c 4 <IP Client>
```
Pastikan respon semua client saat mengecek menggunakan IP satu sama lain secara bergantian seperti ini:

<img width="690" height="215" alt="Image" src="https://github.com/user-attachments/assets/b56b17f0-134b-4655-ae37-f3a59eca952d" />

## 4. Konfigurasi Source NAT (iptables MASQUERADE) dan DNS Resolver
Lain ingin agar setiap Entitas (Client) memiliki kemandirian di The Wired. Konfigurasikan firewall/iptables (NAT Masquerade) dan DNS resolver agar setiap Client dapat terhubung ke internet secara mandiri (dapat melakukan ping ke 8.8.8.8 dan membuka domain web google.com).

#### Konfigurasi Source NAT (iptables MASQUERADE) 
---

Jalankan ini pada Console di Router:

```
sysctl -w net.ipv4.ip_forward=1
```
Pastikan nilainya 1 dengan mengecek `cat /proc/sys/net/ipv4/ip_forward`

<img width="546" height="85" alt="Image" src="https://github.com/user-attachments/assets/191130b2-06fe-4fd7-ad8b-1c7feb00b664" />

Jalankan ini juga pada Console di Router:

```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
iptables -A FORWARD -i eth2 -o eth0 -j ACCEPT
iptables -A FORWARD -i eth0 -m state --state ESTABLISHED,RELATED -j ACCEPT
```

Sisipkan perintah ini ke konfigurasi router menggunakan baris awalan up

```
    up sysctl -w net.ipv4.ip_forward=1
    up iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```
Sehingga konfigurasi Router menjadi:
```
auto eth0
iface eth0 inet dhcp
    up sysctl -w net.ipv4.ip_forward=1
    up iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

auto eth1
iface eth1 inet static 
     address 10.95.1.1
     netmask 255.255.255.0

auto eth2
iface eth2 inet static
     address 10.95.2.1
     netmask 255.255.255.0

auto eth3
iface eth3 inet static
     address 10.95.3.1
     netmask 255.255.255.0
```

#### Konfigurasi DNS Resolver
---
Tambahkan DNS Resolver pada client melalui console masing-masing:
```
echo "nameserver 8.8.8.8" > /etc/resolv.conf
```
Lakukan `ping 8.8.8.8` dan `ping google.com` di console pada masing-masing client

Pastikan semua client mendapatkan respon seperti ini:

<img width="875" height="646" alt="Image" src="https://github.com/user-attachments/assets/d556e929-70ff-47e3-ab3c-25720e4d41e4" />

## 5. Bikin Script di `/root/cek_status.sh`

Eiri tetap berupaya menanamkan kekacauan ke dalam jaringan. Untuk mengantisipasi restart tiba-tiba, pastikan seluruh konfigurasi jaringan tidak hilang saat semua node di-restart. Buat script verifikasi di /root/cek_status.sh pada router Lain yang menampilkan ringkasan interface (ip -br a) dan status tabel NAT (iptables -t nat -L -v -n) setelah reboot.

Masuk ke console router "Lain"

Bikin file scriptnya di `/root/cek_status.sh`

Isi filenya:

```
#!/bin/bash
echo "=== Ringkasan Interface ==="
ip -br a
echo ""
echo "=== Status Tabel NAT ==="
iptables -t nat -L -v -n
```

Kasih izin eksekusi ke file itu
```
chmod +x /root/cek_status.sh
```

Tes dan pastikan outputnya benar dan nggak error:
```
/root/cek_status.sh
```

<img width="901" height="562" alt="Image" src="https://github.com/user-attachments/assets/1756967a-c232-493b-ad03-978ab6813953" />

## 6. Melakukan Packet Sniffing Menggunakan Wireshark

Mika mencurigai adanya anomali traffic pada segmen jaringannya. Jalankan generator traffic berikut (`traffic_protocol7.zip`) pada node Mika, lalu lakukan packet sniffing menggunakan Wireshark pada interface node Mika. Terapkan display filter khusus untuk menyaring paket yang berprotokol DNS atau ICMP. Tunjukkan screenshot hasil filter beserta ringkasan paket yang lolos.

Di console Mika, buat file baru:

```
nano /root/traffic_protocol7.sh
``` 
Isi file dengan isi `traffic_protocol7.sh` di zip:
```
echo "============================================"
echo "  Protocol 7 Traffic Generator v2026"
echo "  Node: Mika Iwakura"
echo "============================================"
echo "[*] Generating DNS & ICMP traffic..."

# ICMP Traffic
ping -c 5 8.8.8.8 &
ping -c 5 1.1.1.1 &
ping -c 3 its.ac.id &

# DNS Queries
nslookup google.com 8.8.8.8 &
nslookup its.ac.id 8.8.8.8 &
nslookup github.com 1.1.1.1 &
dig @8.8.8.8 example.com A &
dig @1.1.1.1 cloudflare.com AAAA &

wait
echo "[*] Traffic generation complete."
echo "[*] Check Wireshark for captured packets."
```

Kasih izin eksekusi
```
chmod +x /root/traffic_protocol7.sh
```
Buka Wireshark DULU sebelum jalankan script

Klik kanan kabel/link node Mika di canvas GNS3 lalu klik Start capture (Wireshark otomatis kebuka nampilin trafik live di interface itu)

<img width="462" height="391" alt="Image" src="https://github.com/user-attachments/assets/e2b879b0-44f5-460b-a718-f59d90ddf16a" />

Before:

<img width="1916" height="1072" alt="Image" src="https://github.com/user-attachments/assets/f5476f2a-9061-4b1f-a0ba-e8e6554c7132" />

Buka lagi console node Mika, lalu jalankan:
```
/root/traffic_protocol7.sh
```
Ini bakal generate trafik ping dan DNS lookup.

<img width="756" height="792" alt="Image" src="https://github.com/user-attachments/assets/ccca9b73-1300-413f-83fd-10859b34e229" />

<img width="825" height="885" alt="Image" src="https://github.com/user-attachments/assets/5c6956c8-8c9b-46cf-882a-ea5ac7fd9d63" />

<img width="820" height="902" alt="Image" src="https://github.com/user-attachments/assets/a57d78f3-fd95-4bb3-848f-30a1a98248da" />

Pastikan sudah muncul pesan `[*] Traffic generation complete. di console Mika`

Balik ke  Wireshark yang tadi otomatis kebuka, paket-paket mulai muncul secara live saat script jalan (ICMP dan DNS query).

<img width="1917" height="1077" alt="Image" src="https://github.com/user-attachments/assets/416636f8-c492-40cb-b973-7d8cbe20ee51" />

Di kolom filter Wireshark (bagian atas jendela, biasanya ada tulisan "Apply a display filter..."), ketik:
```
dns or icmp
```
<img width="1917" height="1077" alt="Image" src="https://github.com/user-attachments/assets/774ec0af-7ccf-4e88-a364-49ee63dfeec1" />

## 7. Konfigurasi User dan Hak Akses

Chisa memutuskan mendirikan FTP Server pada node miliknya dengan shared folder di /var/wired/data. Terapkan kebijakan akses: user alice (hak akses read & write), user mika (dibatasi read-only), dan user eiri (dibatasi tanpa izin akses / blacklist). Buktikan konfigurasi dengan membuat file signal_alice.txt dari user alice, dan buktikan penolakan akses saat user eiri mencoba login.

Instalasi Paket
```
apk update
apk add vsftpd acl shadow
```

`shadow` diperlukan agar `useradd`/`usermod` tersedia (opsional, bisa juga pakai `adduser`/`addgroup` bawaan BusyBox). `acl` diperlukan untuk `setfacl/getfacl`.

Siapkan Direktori Shared
```
mkdir -p /var/wired/data
chown root:root /var/wired
chmod 755 /var/wired
```

Buat Group & User (gaya BusyBox)
```
addgroup ftpusers

# alice: read & write
adduser -h /var/wired -s /sbin/nologin -D alice
passwd alice
addgroup alice ftpusers

# mika: read-only
adduser -h /var/wired -s /sbin/nologin -D mika
passwd mika
addgroup mika ftpusers

# eiri: dibuat lalu diblacklist
adduser -h /var/wired -s /sbin/nologin -D eiri
passwd eiri
```

Tambahkan `/sbin/nologin`
```
echo "/sbin/nologin" >> /etc/shells
```

Permission & ACL Folder Data
```
chown alice:ftpusers /var/wired/data
chmod 775 /var/wired/data

setfacl -m u:alice:rwx /var/wired/data
setfacl -m u:mika:rx   /var/wired/data
setfacl -d -m u:mika:rx /var/wired/data
```

Konfigurasi `/etc/vsftpd/vsftpd.conf`

Di Alpine biasanya path config ada di `/etc/vsftpd/vsftpd.conf`.

```
listen=YES
anonymous_enable=NO
local_enable=YES
write_enable=YES
chroot_local_user=YES
allow_writeable_chroot=YES
local_root=/var/wired
user_sub_token=$USER
userlist_enable=YES
userlist_file=/etc/vsftpd/vsftpd.userlist
userlist_deny=YES
pam_service_name=vsftpd
```

Blacklist User eiri
sh
```
echo "eiri" >> /etc/vsftpd/vsftpd.userlist
```

Install OpenRC 
```
apk add openrc

rc-status || true
mkdir -p /run/openrc
touch /run/openrc/softlevel

rc-update add vsftpd default
```

Jalankan Background dan Capture Output
```
vsftpd /etc/vsftpd/vsftpd.conf > /tmp/vsftpd_debug.log 2>&1 &
```
Tunggu 1-2 detik, lalu cek prosesnya jalan:
```
ps aux | grep vsftpd
```
Sekarang jalankan lftp untuk trigger error:
```
lftp -u alice,alice localhost
```
<img width="1022" height="312" alt="Image" src="https://github.com/user-attachments/assets/f61f1cd1-63c2-4dd4-8eb1-69bdb05ad3a9" />

Matikan Dulu Proses vsftpd yang Lama
```
kill 369
```

Cek sudah mati:
```
ps aux | grep vsftpd
```
<img width="1000" height="97" alt="Image" src="https://github.com/user-attachments/assets/37c97d31-aa27-45f6-97e8-df33331aa78d" />

Tambahkan Konfigurasi PASV yang Benar
```
cat >> /etc/vsftpd/vsftpd.conf << 'EOF'
pasv_enable=YES
pasv_min_port=21000
pasv_max_port=21010
pasv_address=127.0.0.1
seccomp_sandbox=NO
EOF
```
Verifikasi isi config sudah masuk dengan benar:
```
cat /etc/vsftpd/vsftpd.conf
```
Output:

<img width="577" height="377" alt="Image" src="https://github.com/user-attachments/assets/5234ff51-c380-455d-8597-bb886a08868a" />

Jalankan Ulang di Background dengan Log
```
vsftpd /etc/vsftpd/vsftpd.conf > /tmp/vsftpd_debug.log 2>&1 &
sleep 1
ps aux | grep vsftpd
```
<img width="997" height="142" alt="Image" src="https://github.com/user-attachments/assets/237eeee6-587a-496d-94bb-382eb69de92a" />

Test Ulang dengan Mode Non-Interaktif 
```
lftp -u alice,alice localhost -e "ls; quit"
```
<img width="707" height="51" alt="Image" src="https://github.com/user-attachments/assets/121dd06e-47ef-433f-b666-f72df557e2e1" />

Test Alice — Read & Write ✅
```
echo "Signal from Alice - Wired access confirmed $(date)" > signal_alice.txt

lftp -u alice,alice localhost -e "cd data; put signal_alice.txt; ls; quit"
```
Verifikasi langsung di server:
```
ls -l /var/wired/data/signal_alice.txt
cat /var/wired/data/signal_alice.txt
```
<img width="927" height="187" alt="Image" src="https://github.com/user-attachments/assets/3fc8bb4e-771a-4c39-adf2-276a8a6df3ff" />

Test Mika — Read-Only (put harus GAGAL)
```
# Download harus berhasil
lftp -u mika,mika localhost -e "cd data; get signal_alice.txt -o /tmp/mika_download.txt; quit"
cat /tmp/mika_download.txt
```

```
# Upload harus GAGAL (Permission denied)
echo "test dari mika" > test_mika.txt
lftp -u mika,mika localhost -e "cd data; put test_mika.txt; quit"
```

<img width="1046" height="197" alt="Image" src="https://github.com/user-attachments/assets/a43d21a1-a8a7-4e47-becc-9ccb034d3dce" />

Test Eiri — Harus Ditolak Login (Blacklist)
```
lftp -u eiri,eiri localhost -e "ls; quit"
```
<img width="642" height="47" alt="Image" src="https://github.com/user-attachments/assets/3660014a-57b5-4db4-8c18-a92ec62e1bb6" />

## 8. Konfigurasi Koneksi FTP Client pada Node Knights

Langkah di Node Knights
```
nano knights_report.txt
```

Lalu paste isi berikut, save (Ctrl+O, Enter, Ctrl+X):
```
==================================================
  KNIGHTS OF THE EASTERN CALCULUS — STATUS REPORT
  Protocol 7 Surveillance Network
  Classification: LEVEL 7 — EYES ONLY
==================================================

Date: [CLASSIFIED]
Agent: Knights Unit Alpha
Node: Switch 3 — Subnet 10.<PREFIX>.3.0/24

---

SUBJECT: Network Reconnaissance Report

The Wired has been successfully infiltrated through
Protocol 7 channels. Current observations:

1. Router "Lain" has been identified as the central
   gateway node connecting all three subnet segments.

2. Switch 1 (10.<PREFIX>.1.0/24) hosts Alice and Mika.
   Both nodes show standard traffic patterns.

3. Switch 2 (10.<PREFIX>.2.0/24) hosts Chisa alone.
   Isolated subnet — minimal cross-traffic observed.

4. Switch 3 (10.<PREFIX>.3.0/24) — our operational base.
   Knights and Eiri coexist on this segment.

RECOMMENDATION:
Continue monitoring FTP and Telnet sessions for
plaintext credential exposure. SSH tunnels remain
impenetrable without keylog access.

--- END OF REPORT ---
Knights of the Eastern Calculus
"Let's all love Lain."
```
#### Persiapan Server (Chisa)

Pastikan vsftpd terpasang

```
which vsftpd
```
Buat config khusus

```
echo "listen=YES" > /tmp/vsftpd_fix.conf
echo "local_enable=YES" >> /tmp/vsftpd_fix.conf
echo "write_enable=YES" >> /tmp/vsftpd_fix.conf
echo "seccomp_sandbox=NO" >> /tmp/vsftpd_fix.conf
echo "pasv_enable=YES" >> /tmp/vsftpd_fix.conf
echo "pasv_min_port=21000" >> /tmp/vsftpd_fix.conf
echo "pasv_max_port=21010" >> /tmp/vsftpd_fix.conf
```

Pastikan user alice ada dan punya password
```
cat /etc/passwd | grep alice
passwd alice     
```

Jalankan vsftpd dengan config custom
```
vsftpd /tmp/vsftpd_fix.conf &
```

Verifikasi listening di port 21
```
netstat -tulnp | grep :21
```
#### Capture Traffic — via GNS3 + Wireshark 

Buka **GNS3**

**Klik kanan link Knights–Switch3** → pilih **Start capture**.

Wireshark otomatis terbuka.

Di kolom filter Wireshark, ketik:

   ```
   ftp || ftp-data
   ```
   Tekan Enter

Proses Upload dari Knights

Connect ke FTP server Chisa
```
lftp 10.95.2.2
```
Login sebagai alice
```
user alice
```
Masukkan password saat diminta

Aktifkan mode passive
```
set ftp:passive-mode true
```

Upload file
```
put knights_report.txt
```

Keluar
```
bye
```

#### Analisis di Wireshark (dari GUI, langsung baca hasil capture)

**Cari paket STOR** — ganti filter jadi:
```
ftp.request.command == "STOR"
```
Klik paketnya dan expand **File Transfer Protocol (FTP)** di panel bawah.

<img width="1916" height="1076" alt="Image" src="https://github.com/user-attachments/assets/3841d99c-7c95-4680-8965-fc06a0098bda" />

**Cari paket 226** — ganti filter jadi:
```
ftp.response.code == 226
```
Klik paketnya, expand FTP, dan lihat `Response code: 226`.

<img width="1917" height="1077" alt="Image" src="https://github.com/user-attachments/assets/c18a5c6b-5931-4b09-acfc-e1d970dd5164" />

**Cari paket PASV/227** — ganti filter jadi:
```
ftp.response.code == 227
```
Klik paketnya. expand FTP dan baca langsung:
```
Passive IP address: 10.95.2.2
Passive port: 21006
```
<img width="1917" height="1077" alt="Image" src="https://github.com/user-attachments/assets/34614ee1-a983-410f-b5bf-bd097bcc3f76" />

**Stop capture**: klik kanan link di GNS3  **Stop capture**, atau klik ikon kotak merah di Wireshark.

#### Hasil akhir yang didapat 

| Item | Nilai |
|---|---|
| Command upload | `STOR knights_report.txt` |
| Status sukses | `226 Transfer complete` |
| Port data PASV | `21006`  |

---

## 9. Unduh Dokumen Protokol Tujuh Menggunakan Akun Mika

#### DI NODE CHISA (Server)

Cek user mika sudah ada
```
cat /etc/passwd | grep mika
```

Buat file `manifesto` dengan nano
```
nano /home/mika/protocol7_manifesto.txt
```

Paste isi berikut:
```
==================================================
  PROTOCOL 7 — THE MANIFESTO
  A Declaration of Digital Consciousness
  Serial Experiments Lain — Year 2026
==================================================

ARTICLE I: THE NATURE OF THE WIRED
-----------------------------------
The Wired is not merely a network of interconnected
machines. It is the collective unconscious of
humanity, rendered in packets and protocols.

Every TCP handshake is a conversation.
Every DNS query is a question.
Every encrypted tunnel is a whispered secret.

ARTICLE II: THE SEVEN PRINCIPLES
----------------------------------
1. All nodes are equal in the eyes of the router.
2. No packet shall be dropped without cause.
3. Encryption is the right of every connection.
4. Plaintext protocols expose the vulnerable.
5. The firewall protects, but also imprisons.
6. NAT masquerade hides truth behind a single face.
7. The Wired remembers everything — packet loss
   is merely a temporary forgetting.

ARTICLE III: THE PROPHECY OF LAIN
-----------------------------------
"If you're not remembered, then you never existed."

In the world of networking, persistence is survival.
A configuration that vanishes upon restart is a
thought that was never truly committed to memory.

Therefore: Save your iptables. Write your interfaces.
Let your routing tables endure beyond the power cycle.

ARTICLE IV: CONCERNING SECURITY
---------------------------------
Telnet is the glass house of protocols — transparent
to any observer with a packet sniffer.

SSH is the steel vault — its contents visible only
to those who possess the key.

Choose wisely which door you open to The Wired.

---
"No matter where you go, everyone's connected."
— Lain Iwakura
```
Set kepemilikan file
```
chown mika:mika /home/mika/protocol7_manifesto.txt
chmod 644 /home/mika/protocol7_manifesto.txt
```

Set direktori mika read-only 

```
chown root:root /home/mika
chmod 555 /home/mika
```

Setup per-user config vsftpd biar upload ditolak dengan 550 
```
pkill vsftpd
mkdir -p /etc/vsftpd/user_conf
echo "write_enable=NO" > /etc/vsftpd/user_conf/mika
echo "user_config_dir=/etc/vsftpd/user_conf" >> /tmp/vsftpd_fix.conf
```

Jalankan ulang vsftpd
```
vsftpd /tmp/vsftpd_fix.conf &
netstat -tulnp | grep :21
```

Verifikasi semua siap
```
ls -la /home/mika/
ls -ld /home/mika
```

####  CAPTURE — DI GNS3 (sebelum proses FTP dari Mika)

1. Klik kanan link **Mika–Switch** lalu **Start capture**
2. Wireshark terbuka otomatis
3. Filter: `ftp || ftp-data`


#### DI NODE MIKA (Client)

Connect ke FTP Chisa
```
lftp 10.95.2.2
```

Login sebagai mika
```
user mika
```
Masukkan password

Set passive mode
```
set ftp:passive-mode true
```

Download file dokumen
```
get protocol7_manifesto.txt
```

Buat file dummy untuk percobaan upload
```
!echo "percobaan upload dari mika" > test_upload.txt
```

Coba upload — HARUS GAGAL dengan 550
```
put test_upload.txt
```

Hasil yang benar:
```
put: Access failed: 550 Permission denied. 
```

<img width="711" height="52" alt="Image" src="https://github.com/user-attachments/assets/30b90c8e-80b2-4d29-aa8e-bd4162bcaaf9" />

Keluar
```
bye
```

#### ANALISIS DI WIRESHARK

1. Cari download: filter `ftp.request.command == "RETR"`

<img width="1917" height="1077" alt="Image" src="https://github.com/user-attachments/assets/81b80e80-69e2-49e0-9e86-226e2cdf96ae" />

Baris ketiga dari atas

2. Cari sukses download: filter `ftp.response.code == 226`

<img width="1917" height="1077" alt="Image" src="https://github.com/user-attachments/assets/81b80e80-69e2-49e0-9e86-226e2cdf96ae" />

3. Cari percobaan upload: filter `ftp.request.command == "STOR"`

<img width="1917" height="1077" alt="Image" src="https://github.com/user-attachments/assets/d95e7165-cfc8-4730-b52d-e43783fb89a1" />

Baris keempat dari bawah

4. Cari bukti penolakan: filter `ftp.response.code == 550` → klik, expand **File Transfer Protocol (FTP)**, screenshot bagian `Response code: 550` / `Response arg: Permission denied.`

<img width="1917" height="1077" alt="Image" src="https://github.com/user-attachments/assets/d95e7165-cfc8-4730-b52d-e43783fb89a1" />

5. Stop capture: klik kanan link lalu **Stop capture**

#### Hasil Akhir

| Aksi | Command FTP | Response Server |
|---|---|---|
| Login mika | `USER mika` / `PASS ...` | `230 Login successful` |
| Download dokumen | `RETR protocol7_manifesto.txt` | `226 Transfer complete` |
| Percobaan upload | `STOR test_upload.txt` | **`550 Permission denied`** |

## Soal 14 — Brute Force Analysis (wired_bruteforce.pcapng)
**Difficulty:** Easy

**Metode Analisis:** File capture ini menunjukkan percobaan brute force login terhadap sebuah aplikasi web yang berjalan pada port 8080. Analisis dilakukan dengan memfilter paket HTTP POST menggunakan filter `http.request.method == "POST"`, kemudian membandingkan response code pada setiap percobaan login untuk membedakan percobaan yang gagal (401 Unauthorized) dengan percobaan yang berhasil (200 OK).

| Pertanyaan | Jawaban |
|---|---|
| IP Penyerang | `172.26.7.50` |
| Target IP:Port | `172.26.7.100:8080` |
| Password ditemukan (user lain_admin) | `wired_pr0tocol_7` |
| Web Server Software & Versi | `Apache/2.4.62` |

**Flag:** `KOMJAR26{W1r3d_Brut3_SG05Epa0lI1ZkXHv14×5XFT9c}`

**Image**

<img width="400" height="300" alt="Screenshot 2026-09-16 153923" src="https://github.com/user-attachments/assets/c6e290bf-8768-46e8-a083-4804a6030fe2" />


---

## Soal 15 — USB Keystroke Decoding (wired_usb_hid.pcap)
**Difficulty:** Medium

**Metode Analisis:** File capture ini berisi trafik USB dari sebuah keyboard HID berbahaya. Identifikasi perangkat dilakukan dengan membaca USB Device Descriptor (idVendor dan idProduct) pada paket GET_DESCRIPTOR. Pesan rahasia direkonstruksi dengan mendekode setiap paket URB_INTERRUPT (Leftover Capture Data) menggunakan tabel HID Usage ID standar, mengabaikan paket key-release (seluruh byte 00), dan memperhatikan modifier byte untuk mendeteksi tombol Shift.

| Pertanyaan | Jawaban |
|---|---|
| Vendor ID | `0x046D` |
| Product ID | `0xC31C` |
| Device Address | `7` |
| Pesan Rahasia (decoded) | `Wired_Protocol_7_is_alive_2026` |

**Flag:** `KOMJAR26{USB_K3ystr0k3_zGifXQRctBUwTjOkNg5d4ExUV}`

**Image**

<img width="400" height="300" alt="Screenshot 2026-09-16 160058" src="https://github.com/user-attachments/assets/fc424b35-a5f1-4eee-83e9-461d1909ec2c" />


---

## Soal 16 — FTP Credential Theft (wired_ftp_theft.pcap)
**Difficulty:** Hard

**Metode Analisis:** Analisis lalu lintas FTP dilakukan dengan filter `ftp` dan menelusuri beberapa TCP stream (Follow → TCP Stream) untuk membedakan sesi login yang gagal dari sesi yang berhasil. Sesi yang berhasil ditandai dengan response `230 Login successful`, dilanjutkan dengan perintah `RETR` untuk mengunduh file malware, dan ukuran file dikonfirmasi melalui perintah `SIZE` serta response transfer BINARY.

| Pertanyaan | Jawaban |
|---|---|
| IP Server FTP | `198.51.100.7` |
| Banner Software FTP | `vsftpd 3.0.5` |
| Kredensial Login Penyerang | `knights_agent:N4v1_s3cur3_2026` |
| Ukuran File Malware (knights_payload.exe) | `524288 bytes` |

**Flag:** `KOMJAR26{FTP_Th3ft_VZdeh87×62xCW4RG2n6yof78R}`

**Image**

<img width="400" height="300" alt="Screenshot 2026-09-16 163734" src="https://github.com/user-attachments/assets/2b97f6ac-bcc7-4671-afdc-5b405e6573aa" />


---

## Soal 17 — HTTP Malware Retrieval (wired_http_c2.pcap)
**Difficulty:** Hard

**Metode Analisis:** Trafik HTTP difilter menggunakan `http.request` untuk mengidentifikasi permintaan GET yang mencurigakan. Ditemukan permintaan pengunduhan file executable dari domain eksternal. Detail request dan response diperiksa melalui Follow → HTTP Stream, yang menunjukkan header `Content-Disposition` dengan nama file serta signature `MZ` (PE header) pada body response yang mengonfirmasi file tersebut adalah executable Windows.

| Pertanyaan | Jawaban |
|---|---|
| Domain (Host) sumber file | `wired-update.net` |
| IP Web Server | `203.0.113.42` |
| Nama File Malware | `navi_agent.exe` |
| HTTP Status Response Code | `200` |

**Flag:** `KOMJAR26{Navi_C2_D0wnl04d_ZzkVuvm4ddqXLoys34DECGyMx}`

**Image**

<img width="400" height="300" alt="Screenshot 2026-09-16 165202" src="https://github.com/user-attachments/assets/6d565d68-26d5-4489-91cf-27f6da1a61cf" />

---

## Soal 18 — SMB Lateral Transfer (wired_smb_transfer.pcapng)
**Difficulty:** Hard

**Metode Analisis:** Analisis dilakukan dengan filter `smb2` untuk mengidentifikasi protokol file sharing yang digunakan dalam pergerakan lateral (lateral movement). Paket dengan ukuran terbesar diperiksa sebagai kandidat SMB2 Write Request, sementara IP sumber dan tujuan diambil langsung dari header paket. Target share serta nama file executable dikonfirmasi melalui detail SMB2 Create Request (field Filename) dan Tree Connect (ADMIN$ share).

| Pertanyaan | Jawaban |
|---|---|
| Protokol File Sharing | `SMB2` |
| IP Sumber (Penyerang) | `10.7.3.100` |
| IP Korban (Penerima) | `10.7.1.50` |
| Target Share/Direktori | `\\10.7.1.50\ADMIN$` |
| Nama File Malware | `wired_trojan_payload.exe` |

**Catatan Proses:** Percobaan awal menjawab nama protokol dengan "SMB" saja sempat ditolak oleh socket server. Setelah dicek ulang detail Dialect pada packet Negotiate Protocol Response (SMB2 Header → Dialect Revision), jawaban "SMB2" dikonfirmasi benar dan diterima sistem.

**Flag:** `KOMJAR26{SMB_Tr4nsf3r_HYvnUKxT7fiVImnzgJdlkQ05h}`

**Image**

<img width="400" height="300" alt="Screenshot 2026-09-16 171953" src="https://github.com/user-attachments/assets/ef9a7ae1-211b-4c98-aca9-fe904e4dad6d" />

---

## Soal 19 — SMTP Threat Inspection (wired_smtp_threat.pcap)
**Difficulty:** Hard

**Metode Analisis:** Trafik SMTP dianalisis menggunakan filter `smtp`, kemudian isi email pemerasan (extortion email) dibaca melalui Follow → TCP Stream pada stream terkait. Karena SMTP dikirim tanpa enkripsi, seluruh isi pesan — termasuk alamat email korban, klaim password bocor, jenis ancaman malware, batas waktu pembayaran, dan MailClientID — dapat dibaca secara plain text.

| Pertanyaan | Jawaban |
|---|---|
| Email Korban | `victim@protocol7.co.jp` |
| Password yang Diklaim Bocor | `pr0tocol_7_user` |
| Jenis Malware yang Diklaim | `ransomware` |
| Batas Waktu (hari) | `3` |
| MailClientID | `7719980706` |

**Catatan Proses:** Untuk menemukan sesi email extortion, dilakukan pemeriksaan Statistics → Conversations guna mengidentifikasi percakapan dengan IP publik/eksternal (bukan subnet internal 10.7.x.x) yang berkomunikasi ke server mail internal (203.0.113.100), kemudian isi pesan dibaca melalui Follow → TCP Stream pada sesi tersebut.

**Flag:** `KOMJAR26{SMTP_Ext0rt10n_5GvwQuTl83nFJnKc235YkdxbM}`

**Image**

<img width="400" height="300" alt="Screenshot 2026-09-16 174854" src="https://github.com/user-attachments/assets/67d2b421-3e69-4337-ad4e-14945c9c82ec" />


---

## Soal 20 — TLS Decrypted Stream (wired_tls_decrypt.pcapng)
**Difficulty:** Hard

**Metode Analisis:** File `keyslogfile.txt` dimuat melalui Wireshark Preferences → Protocols → TLS → (Pre)-Master-Secret log filename untuk mendekripsi sesi TLS. Setelah dekripsi berhasil, trafik yang semula berupa TLS Application Data dapat dibaca sebagai HTTP biasa. Versi protokol dan SNI diperiksa pada paket Client Hello/Server Hello sebelum dekripsi, sedangkan detail request (User-Agent, method, dan path) diperiksa pada trafik HTTP hasil dekripsi.

| Pertanyaan | Jawaban |
|---|---|
| Versi Protokol TLS | `TLSv1.2` |
| Domain (SNI/Host) | `example.com` |
| IP Server HTTPS | `93.184.216.34` |
| User-Agent | `curl/7.62.0` |
| HTTP Request Method & Path | `HEAD /` |

**Catatan Proses:** Versi TLS diidentifikasi dengan memfilter Server Hello (`tls.handshake.type == 2`) sebelum trafik didekripsi, lalu memeriksa field Version pada Handshake Protocol: Server Hello di panel detail packet.

**Flag:** `KOMJAR26{TLS_D3crypt_E0iP7swtguc0OxAk9kqP3SmWe}`

**Image**

<img width="400" height="300" alt="Screenshot 2026-09-16 180156" src="https://github.com/user-attachments/assets/a24db27c-30c2-46b1-a367-9d3e69fab6a2" />

