# UTS Praktik Infrastruktur & Network Attacks — Kasus 8

Repository ini berisi dokumentasi pengerjaan UTS Praktik Infrastruktur & Network Attacks **Kasus 8**, Pertemuan 1 sampai 7: source lab, panduan praktikum, hasil pengujian, dan screenshot bukti.

## Identitas

| Informasi | Keterangan |
|---|---|
| Nama | Revin Erdiansyah |
| NIM | 24552011387 |
| Kelas | TIF RP 24E CNS |
| Mata Kuliah | Network Security |
| Nomor Kasus | 8 |
| Cakupan Pertemuan | 1–7 |
| Lingkungan | Kali Linux dan Docker |

## Dokumentasi Utama

**[Buka Jawaban UTS Kasus 8](03-hasil-kasus-8/jawaban.md)**

`03-hasil-kasus-8/jawaban.md` adalah laporan utama yang menggabungkan tujuan, perintah, bukti, hasil, dan kesimpulan Tugas 1–7. Folder `tugas-1` sampai `tugas-7` berisi dokumentasi dan screenshot pendukung per tugas.

## Struktur Repository

```text
UTS-Network-Security/
├── README.md
├── 01-sumber-lab/
│   ├── README.md
│   ├── b/        (b1-compose, b3-syn-flood, b4-dns-exfil)
│   ├── bukti/
│   │   └── kasus-8/kirim_icmp.py
│   ├── images/   (uts-openssl, uts-kali-tools, uts-dnsmasq)
│   ├── kasus/    (k5-scapy, k9-dns-detect)
│   ├── cert-k8.pem
│   ├── dns-k8.log
│   ├── hasil-k8.txt
│   ├── laporan-k8.enc
│   └── laporan-k8.txt
├── 02-dokumen/
│   ├── DOKUMEN-UTS-LENGKAP.md
│   └── LEMBAR-PERINTAH-PER-KASUS.md
└── 03-hasil-kasus-8/
    ├── jawaban.md
    ├── tugas-1/
    │   ├── hasil.md
    │   └── screenshot/
    ├── tugas-2/
    │   ├── hasil.md
    │   └── screenshot/
    ├── tugas-3/
    │   ├── hasil.md
    │   └── screenshot/
    ├── tugas-4/
    │   ├── hasil.md
    │   └── screenshot/
    ├── tugas-5/
    │   ├── hasil.md
    │   └── screenshot/
    ├── tugas-6/
    │   ├── hasil.md
    │   └── screenshot/
    └── tugas-7/
        ├── hasil.md
        └── screenshot/
```

| Folder | Keterangan |
|---|---|
| `01-sumber-lab/` | Dockerfile, docker-compose, skrip praktikum dari dosen, serta file keluaran lab Kasus 8 |
| `02-dokumen/` | Dokumen soal dan lembar perintah per kasus |
| `03-hasil-kasus-8/` | Jawaban utama, hasil per tugas, dan screenshot |

## Ringkasan Praktikum

| Pertemuan | Materi | Dokumentasi |
|---|---|---|
| 1 | Docker dan Volume | [tugas-1](03-hasil-kasus-8/tugas-1/hasil.md) |
| 2 | Kriptografi | [tugas-2](03-hasil-kasus-8/tugas-2/hasil.md) |
| 3 | ARP Spoofing | [tugas-3](03-hasil-kasus-8/tugas-3/hasil.md) |
| 4 | Analisis Handshake WPA2 | [tugas-4](03-hasil-kasus-8/tugas-4/hasil.md) |
| 5 | ICMP dengan Scapy | [tugas-5](03-hasil-kasus-8/tugas-5/hasil.md) |
| 6 | SYN Flood dan Three-Way Handshake | [tugas-6](03-hasil-kasus-8/tugas-6/hasil.md) |
| 7 | DNS Exfiltration | [tugas-7](03-hasil-kasus-8/tugas-7/hasil.md) |

## Penentuan Nomor Kasus

Nomor kasus = jumlah digit NIM mod 10 (hasil 0 berarti Kasus 10).

```text
NIM          : 24552011387
Jumlah digit : 2+4+5+5+2+0+1+1+3+8+7 = 38
Perhitungan  : 38 mod 10 = 8
Hasil        : Kasus 8
```

## Persiapan Lingkungan

```bash
docker --version
docker compose version
cd 01-sumber-lab
docker build -t uts-openssl images/uts-openssl
docker build -t uts-kali-tools images/uts-kali-tools
docker build -t uts-dnsmasq images/uts-dnsmasq
docker pull alpine
docker pull nginx:alpine
```

Jaringan Bagian B memakai `internal: true` (tanpa akses internet), sehingga image harus disiapkan sebelum praktikum.

## Catatan Praktikum

1. Seluruh pengujian dilakukan pada lingkungan lab yang terkontrol dan diizinkan.
2. Perintah mengikuti `02-dokumen/LEMBAR-PERINTAH-PER-KASUS.md` untuk Kasus 8.
3. Hasil ditulis berdasarkan keluaran terminal pada screenshot.
4. Perintah yang berhasil dijalankan belum tentu membuktikan serangan berdampak; analisis membedakan proses pengujian dari hasil yang terbukti.

---

**Dokumentasi utama:** [`03-hasil-kasus-8/jawaban.md`](03-hasil-kasus-8/jawaban.md)
