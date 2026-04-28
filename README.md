# Kiw-Kiw-Tech-SOC-Dashboard-Server
A Dockerized, zero-trust Home SOC featuring an air-gapped physical USB key system, real-time threat mapping, and Telegram bot autonomy. Engineered by Kiw Kiw Tech.(still progress)

# 🛡️ Project Antri Gravity: Advanced Home SOC

**Developed by KIW KIW TECH**

> 🚧 **STATUS: STILL IN DEVELOPMENT (WORK IN PROGRESS)**
> Kami berkongsi sistem automasi teras yang telah kami bangunkan khas untuk infrastruktur mikro-server kami sendiri. Projek ini sentiasa berevolusi. Jika anda mempunyai sebarang idea tambahan, maklum balas, atau cadangan fungsi, sila kongsikan! Mari kita bekerjasama untuk membina versi yang paling selamat dan terbaik.

Antri Gravity adalah satu ekosistem Home Security Operations Center (SOC) yang beroperasi sepenuhnya dalam persekitaran Docker yang berprestasi tinggi. Ia direka khusus untuk mengoptimumkan perkakasan kelas enterprise, menampilkan antaramuka hibrid GUI/Terminal, integrasi Telegram Bot untuk pelaksanaan arahan jarak jauh, serta pemetaan risikan siber secara real-time.

---

## 🔐 The "Air-Gapped" Physical USB Key System

**The Problem:** Dashboard berasaskan web secara tradisinya terdedah kepada bot scanner dan serangan brute-force hanya dengan kewujudannya di rangkaian awam (internet).

**The Antri Gravity Solution:** Zero web-facing authentication.
Dashboard ini menggunakan mekanisme **Physical Token Air-Gapped**. Antaramuka web (UI) dashboard akan kekal offline sepenuhnya (memulangkan ralat Fake 404/502) sehinggalah satu pemacu USB fizikal yang telah didaftarkan dimasukkan ke dalam perkakasan server.

### ⚙️ Cara Ia Berfungsi
Kami membangunkan skrip kustom yang menggunakan pengurus peranti kernel Linux (`udev`). Apabila pemacu USB yang sah dikesan, sistem akan mencetuskan Docker untuk menghidupkan container UI secara automatik. Sebaik sahaja USB dicabut secara fizikal, container tersebut akan dimatikan (kill-switch) serta-merta—memberikan tahap keselamatan fizikal yang mutlak.

---

## 🛠️ Panduan Setup: Bina Physical Key Anda

Ikuti langkah ini untuk mengkonfigurasi server anda supaya mengenali pemacu USB tertentu sebagai "Master Key".

### Langkah 1: Kenal Pasti UUID USB Anda
Masukkan pemacu USB pilihan anda ke server. Cari ID uniknya supaya sistem hanya akan dibuka oleh peranti ini sahaja.

```bash
ls -l /dev/disk/by-uuid/
```
*(Cari pemacu USB anda dan salin UUID tersebut, contoh: `A1B2-C3D4`)*

### Langkah 2: Bina Skrip Lock/Unlock
Bina skrip bash untuk mengawal automasi container Docker.

**1. Bina Skrip Unlock (`/usr/local/bin/unlock-soc.sh`):**
```bash
sudo nano /usr/local/bin/unlock-soc.sh
```
*Masukkan kod ini:*
```bash
#!/bin/bash
echo "[$(date)] Master Key Inserted. Unlocking SOC..." >> /var/log/soc-usb.log
docker start antri-gravity-ui
```

**2. Bina Skrip Lock (`/usr/local/bin/lock-soc.sh`):**
```bash
sudo nano /usr/local/bin/lock-soc.sh
```
*Masukkan kod ini:*
```bash
#!/bin/bash
echo "[$(date)] Master Key Removed. Locking SOC..." >> /var/log/soc-usb.log
docker stop antri-gravity-ui
```

**3. Berikan kebenaran 'executable' pada skrip:**
```bash
sudo chmod +x /usr/local/bin/unlock-soc.sh
sudo chmod +x /usr/local/bin/lock-soc.sh
```

### Langkah 3: Konfigurasi Linux UDEV Rules
Arahkan kernel Linux untuk memantau kehadiran USB spesifik anda.

```bash
sudo nano /etc/udev/rules.d/99-soc-usb-key.rules
```
*Masukkan baris berikut (Gantikan `A1B2-C3D4` dengan UUID USB anda):*
```text
# Action on USB INSERT
ACTION=="add", SUBSYSTEM=="block", ENV{ID_FS_UUID}=="A1B2-C3D4", RUN+="/usr/local/bin/unlock-soc.sh"

# Action on USB REMOVE
ACTION=="remove", SUBSYSTEM=="block", ENV{ID_FS_UUID}=="A1B2-C3D4", RUN+="/usr/local/bin/lock-soc.sh"
```

### Langkah 4: Kemas Kini dan Uji
Aktifkan peraturan baru pada sistem:
```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
```

**Ujian:** Cabut USB -> UI akan mati (offline). Masukkan USB 
```
***

 Kiw kiw! 🚀
