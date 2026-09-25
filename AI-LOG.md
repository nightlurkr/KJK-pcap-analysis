
## 1. Alat AI yang Digunakan

| Alat | Peran |
|---|---|
| Claude (Anthropic) | Pendamping analisis: memverifikasi isi PCAP, mengoreksi kesalahan penalaran, dan membantu menyusun laporan |

## 2. Pembagian Kerja

Berbeda dengan Class Activity 02 yang datanya saya hasilkan sendiri lewat simulasi, tugas ini menganalisis PCAP publik. Berikut pemisahan kerjanya.

### Dikerjakan sendiri

- Memilih topik DNS Spoofing dan mencari sumber PCAP publik yang sesuai (`waytoalpit/ManOnTheSideAttack-DNS-Spoofing`).
- Membuka `mycap.pcap` di Wireshark dan melakukan analisis manual: menerapkan filter, membaca panel Packet Details, dan menemukan sendiri bukti utama pada query `www.qq.com` (dua respons dengan DNS ID `0x00e5`, perbedaan MAC dan TTL).
- **Seluruh kelima screenshot adalah hasil tangkapan layar Wireshark saya sendiri**, bukan gambar buatan AI.
- Menyusun draft awal `README.md`, termasuk struktur laporan, tabel bukti, dan pembagian slide PPT.
- Mengelola repositori Git dan proses submission.

### Dibantu AI

- Memverifikasi ulang seluruh isi `mycap.pcap` secara terprogram (parsing 78 frame dengan library Scapy) untuk mengecek apakah draft laporan saya sudah akurat.
- Menemukan dan mengoreksi empat kekurangan pada draft saya (rinciannya di bagian 3).
- Menyunting ulang redaksi `README.md` agar bahasanya lebih mudah dipahami, serta menyisipkan embed gambar yang sebelumnya belum ada.

## 3. Kronologi Interaksi

| # | Yang saya minta / lakukan | Bantuan AI |
|---|---|---|
| 1 | Analisis manual PCAP di Wireshark, lalu menulis draft `README.md` | — (dikerjakan sendiri) |
| 2 | Minta screenshot di-embed ke README | Menyisipkan kelima gambar ke bagian analisis yang relevan dan menghapus catatan status |
| 3 | Minta verifikasi: "apakah isi README sudah sesuai dengan PCAP-nya?" | Mem-parsing ulang seluruh 78 frame, lalu membandingkannya dengan klaim di laporan |

## 4. Verifikasi & Pemahaman

Saya memahami setiap isi laporan ini, termasuk:

- Mengapa TTL 64 mencurigakan: paket dari internet TTL-nya sudah berkurang melewati router, sehingga nilai bulat 64 menandakan paket dibuat di jaringan lokal.
- Mengapa satu IP sumber (`8.8.8.8`) dengan dua MAC address berbeda adalah tanda pemalsuan.
- Perbedaan man-on-the-side dan man-in-the-middle, serta mengapa adanya respons palsu yang kalah cepat justru memperkuat kesimpulan man-on-the-side.
- Apa yang dibuktikan paket ICMP Port unreachable, dan yang lebih penting, apa yang **tidak** dibuktikannya.
- Batasan analisis: capture ini membuktikan upaya DNS spoofing, bukan keberhasilan cache poisoning secara permanen.

## 5. Pernyataan Integritas

PCAP yang dianalisis adalah dataset publik, dan sumbernya telah dicantumkan secara terbuka pada laporan. Analisis awal serta seluruh screenshot Wireshark adalah hasil kerja saya sendiri. AI saya gunakan sebagai alat verifikasi dan penyuntingan, dan setiap koreksi yang diberikannya telah saya periksa ulang sendiri di Wireshark sebelum saya terima. Isi laporan telah saya baca, pahami, dan verifikasi kebenarannya.

**Ryan Adya Purwanto — 5027231046**
