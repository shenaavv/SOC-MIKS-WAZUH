# SOC-MIKS-WAZUH

Project Mini SOC adalah implementasi sistem monitoring keamanan siber berbasis Wazuh SIEM yang dibangun di atas infrastruktur Microsoft Azure. Sistem ini dirancang untuk mengumpulkan, memantau, dan menganalisis log dari beberapa virtual machine secara terpusat, mendeteksi aktivitas mencurigakan seperti serangan DDoS, serta mengirimkan notifikasi keamanan secara real-time melalui Discord.

## Anggota Tim

| No | Nama                   | NRP        |
| -- | ---------------------- | ---------- |
| 1  | Kanafira Vanesha Putri | 5027241010 |
| 2  | Tiara Putri Prasetya   | 5027241013 |
| 3  | Zahra Khaalishah       | 5027241070 |

---

# Link Video Demo

| Demo |
|--------|
| [Record Video Demo](https://drive.google.com/drive/folders/1P1vCq4MT7eb4H11p-H6vzDY3QUO1NUyJ) |

---

# BAB I PENDAHULUAN

## 1.1 Latar Belakang

Ancaman keamanan siber semakin berkembang pesat seiring dengan meningkatnya ketergantungan organisasi terhadap infrastruktur digital. Serangan seperti Distributed Denial of Service (DDoS), brute force authentication, dan eksploitasi layanan web menjadi vektor ancaman yang umum dijumpai di lingkungan produksi maupun akademis. Untuk menghadapi ancaman tersebut, diperlukan suatu sistem pemantauan keamanan yang mampu mengumpulkan, menganalisis, dan merespons insiden secara real-time.

Security Operations Center (SOC) merupakan unit terpusat dalam sebuah organisasi yang bertanggung jawab atas pemantauan, deteksi, dan penanganan insiden keamanan secara berkelanjutan. Pada praktikum ini, dibangun sebuah Mini SOC menggunakan platform open-source Wazuh sebagai solusi Security Information and Event Management (SIEM) yang di-deploy di atas infrastruktur cloud Microsoft Azure.

Arsitektur yang digunakan memanfaatkan tiga Virtual Machine (VM) yang masing-masing menjalankan peran berbeda:

* 1 VM sebagai Wazuh Manager (pusat SOC)
* 1 VM sebagai endpoint yang dimonitor menggunakan Wazuh Agent
* 1 VM sebagai attacker untuk simulasi serangan

Selain itu, sistem dikonfigurasi agar peringatan keamanan tingkat kritikal diteruskan secara otomatis ke Discord melalui webhook sehingga anggota tim dapat menerima notifikasi secara real-time tanpa harus terus memantau dashboard.

## 1.2 Tujuan

* Membangun infrastruktur Mini SOC berbasis Wazuh SIEM di atas cloud Microsoft Azure.
* Mengimplementasikan arsitektur Wazuh Manager-Agent untuk koleksi dan analisis log terdistribusi.
* Mensimulasikan serangan HTTP Flood DDoS menggunakan ApacheBench (ab).
* Mengintegrasikan sistem peringatan Wazuh dengan Discord melalui custom webhook.
* Menganalisis alert yang dihasilkan Wazuh pada dashboard Threat Hunting.

## 1.3 Ruang Lingkup

Praktikum ini mencakup:

* Deployment infrastruktur cloud Azure
* Instalasi dan konfigurasi Wazuh Manager
* Instalasi dan konfigurasi Wazuh Agent
* Monitoring log Nginx
* Simulasi HTTP Flood DDoS
* Analisis alert pada Wazuh Dashboard
* Integrasi notifikasi Discord

---

# BAB II ARSITEKTUR SISTEM

## 2.1 Gambaran Umum Arsitektur

Arsitektur Mini SOC terdiri dari tiga komponen utama yang saling terhubung dalam jaringan virtual Azure. Setiap komponen memiliki fungsi berbeda namun saling melengkapi untuk membentuk alur kerja SOC yang lengkap mulai dari pengumpulan log hingga notifikasi insiden.

## 2.1.1 Komponen Sistem

| VM Name           | Peran                     | Azure Size       | Deskripsi                                            |
| ----------------- | ------------------------- | ---------------- | ---------------------------------------------------- |
| Wazuh-Manager     | SOC / SIEM Server         | Standard_B2s     | Menjalankan Wazuh Manager, Dashboard, dan OpenSearch |
| Agent-Web-Vanesha | Target / Endpoint Monitor | Standard_B2ts v2 | Menjalankan Nginx dan Wazuh Agent                    |
| Agent-Zahra       | Attacker / Agent          | Standard_B2ts v2 | Menjalankan ApacheBench dan Wazuh Agent              |

## 2.1.2 Alur Data dan Komunikasi

```text
Agent-Web-Vanesha
        │
        ▼
     Wazuh Agent
        │
        ▼
   Wazuh Manager
        │
        ▼
    OpenSearch
        │
        ▼
 Wazuh Dashboard
        │
        ▼
 Discord Webhook
```

Port yang digunakan:

| Port     | Fungsi             |
| -------- | ------------------ |
| 22/TCP   | SSH                |
| 80/TCP   | HTTP Nginx         |
| 443/TCP  | HTTPS Dashboard    |
| 1514/TCP | Log Forwarding     |
| 1515/TCP | Agent Registration |

## 2.2 Dokumentasi VM Azure

### Gambar 2.1

Portal Azure menampilkan VM Agent-Zahra dan Agent-Web-Vanesha dalam status Running.

### Gambar 2.2

Detail VM Agent-Web-Vanesha pada Microsoft Azure Portal.

---

# BAB III DEPLOYMENT AZURE

## 3.1 Persiapan Akun Azure for Students

Setiap anggota tim menggunakan akun Azure for Students yang menyediakan kredit sebesar USD 100 untuk penggunaan layanan Azure selama 12 bulan.

Untuk menghemat biaya, VM dihentikan melalui Azure Portal sehingga status berubah menjadi **Stopped (deallocated)** dan biaya komputasi berhenti.

## 3.2 Pembuatan Virtual Machine

### 3.2.1 VM Wazuh-Manager

Spesifikasi:

* Standard_B2s
* 2 vCPU
* 4 GB RAM

Fungsi:

* Wazuh Manager
* OpenSearch
* Wazuh Dashboard

### 3.2.2 VM Agent-Web-Vanesha

Spesifikasi:

* Standard_B2ts v2
* 2 vCPU
* 1 GiB RAM

Fungsi:

* Web Server Nginx
* Target Serangan DDoS
* Wazuh Agent

### 3.2.3 VM Agent-Zahra

Spesifikasi:

* Standard_B2ts v2
* 2 vCPU
* 2 GiB RAM

Fungsi:

* Attacker Machine
* ApacheBench
* Wazuh Agent

## 3.3 Manajemen IP dan SSH

Karena Azure menggunakan Dynamic Public IP, alamat IP dapat berubah setiap kali VM dihidupkan kembali.

Sintaks koneksi:

```bash
ssh username@IP_PUBLIC_VM
```

---

# BAB IV KONFIGURASI WAZUH

## 4.1 Instalasi Wazuh Manager

Dashboard dapat diakses melalui:

```text
https://[IP_PUBLIC_MANAGER]
```

Komponen yang terinstal:

* Wazuh Manager
* OpenSearch
* Wazuh Dashboard

## 4.2 Registrasi dan Instalasi Wazuh Agent

Contoh instalasi agent:

```bash
wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.8.2-1_amd64.deb

sudo WAZUH_MANAGER='72.155.89.170' dpkg -i ./wazuh-agent_4.8.2-1_amd64.deb

sudo systemctl daemon-reload
sudo systemctl enable --now wazuh-agent
```

## 4.3 Konfigurasi Monitoring Log Nginx

```xml
<localfile>
    <log_format>syslog</log_format>
    <location>/var/log/nginx/access.log</location>
</localfile>
```

Restart agent:

```bash
sudo systemctl restart wazuh-agent
```

---

# BAB V LOGGING DAN MONITORING

## 5.1 Mekanisme Koleksi Log

Pipeline Wazuh:

1. Decoder
2. Rules Engine
3. Alert Generation
4. Integration Trigger

## 5.2 Dashboard Overview

Menampilkan:

* Active Agents
* Severity Alerts
* Alert Distribution

## 5.3 Threat Hunting Dashboard

Menampilkan:

* Event Timeline
* Alert Evolution
* MITRE ATT&CK Mapping

---

# BAB VI SIMULASI DDOS / PROOF OF CONCEPT

## 6.1 Instalasi Nginx

```bash
sudo apt install nginx -y
```

## 6.2 ApacheBench

```bash
sudo apt install apache2-utils -y
```

Parameter:

| Flag | Nilai  |
| ---- | ------ |
| -n   | 100000 |
| -c   | 1000   |

## 6.3 Eksekusi Serangan

### HTTP Flood

```bash
ab -n 100000 -c 1000 http://IP_TARGET/
```

### 404 Flood

```bash
ab -n 50000 -c 500 http://IP_TARGET/halaman-palsu.html
```

---

# BAB VII DETECTION DAN ALERT ANALYSIS

## 7.1 Deteksi Alert

Alert utama:

* Rule 31101
* Rule 31151

## 7.2 Analisis Alert

| Rule  | Level | Keterangan                    |
| ----- | ----- | ----------------------------- |
| 31101 | 5     | Web Server 400 Error          |
| 31151 | 10    | Multiple Web Server 400 Error |
| 5902  | 3     | PAM Session Closed            |
| 5710  | 5     | SSH Login Non-existent User   |
| 5501  | 5     | PAM Login Failed              |
| 519   | 7     | Rootcheck Event               |

## 7.3 Expanded Event Detail

Informasi yang tersedia:

* Source IP
* URL
* HTTP Method
* User Agent
* Status Code

## 7.4 Event Timeline

Timeline menampilkan:

* DDoS Activity
* SSH Brute Force
* Authentication Failure
* Agent Status Change

---

# BAB VIII INTEGRASI ALERT DISCORD

## 8.1 Arsitektur Integrasi

```text
Wazuh Alert
      │
      ▼
Custom Script
      │
      ▼
Discord Webhook
      │
      ▼
#wazuh-alerts
```

## 8.2 Custom Integration Script

Konfigurasi:

```xml
<integration>
    <name>custom-discord</name>
    <hook_url>DISCORD_WEBHOOK</hook_url>
    <alert_format>json</alert_format>
    <level>5</level>
</integration>
```

Restart:

```bash
sudo systemctl restart wazuh-manager
```

## 8.3 Hasil Notifikasi

Alert yang berhasil dikirim:

* PAM Login Failed
* SSH Authentication Failed
* Rootcheck Alert
* DDoS Detection Alert

---

# BAB IX KENDALA DAN SOLUSI

## 9.1 Notifikasi Discord Tidak Masuk

**Solusi:** Membuat custom integration script.

## 9.2 Error 50027 Discord Webhook

**Solusi:** Memperbaiki webhook URL dan permission script.

## 9.3 DDoS Tidak Terdeteksi

**Solusi:** Mengurangi concurrent connection dari 1000 menjadi 500.

## 9.4 Perubahan Public IP

**Solusi:** Memeriksa IP terbaru sebelum konfigurasi dan SSH.

---

# BAB X KESIMPULAN

## 10.1 Kesimpulan

* Mini SOC berhasil dibangun menggunakan Wazuh SIEM.
* Sistem mampu mendeteksi HTTP Flood DDoS.
* Integrasi Discord berjalan dengan baik.
* Monitoring dan analisis log berjalan real-time.
* Sistem juga mendeteksi ancaman nyata seperti SSH brute force.

## 10.2 Saran Pengembangan

* Menggunakan Static Public IP.
* Integrasi VirusTotal atau MISP.
* Menambahkan simulasi SQL Injection.
* Menambahkan Port Scanning Detection.
* Mengimplementasikan Wazuh Active Response.

---

# DAFTAR REFERENSI

Wazuh, Inc. (2024). *Wazuh Documentation - Getting Started*. Retrieved from  
https://documentation.wazuh.com/current/getting-started/index.html

Microsoft Azure. (2024). *Azure Virtual Machines Documentation*. Retrieved from  
https://docs.microsoft.com/en-us/azure/virtual-machines/

Apache Software Foundation. (2023). *ab - Apache HTTP Server Benchmarking Tool*. Retrieved from  
https://httpd.apache.org/docs/2.4/programs/ab.html

Discord. (2024). *Webhooks - Discord Developer Documentation*. Retrieved from  
https://discord.com/developers/docs/resources/webhook

Nginx. (2024). *Nginx Access Log Documentation*. Retrieved from  
https://nginx.org/en/docs/http/ngx_http_log_module.html

Wazuh, Inc. (2024). *Wazuh Rules Documentation - Web Attacks*. Retrieved from  
https://documentation.wazuh.com/current/user-manual/ruleset/getting-started.html
