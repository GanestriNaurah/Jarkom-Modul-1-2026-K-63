# Jarkom-Modul-1-2026-K-63

| Nama | NRP |
| :---: | :---: |
| Nadya Putri Agustin | 5027251013 |
| Ganestri Naurah Sawestri | 5027251014 |

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

