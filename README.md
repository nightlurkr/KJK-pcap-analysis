# Analisis DNS Spoofing pada `mycap.pcap` Menggunakan Wireshark

## Identitas kelompok

| Nama | NRP |
|---|---:|
| Ryan Adya Purwanto | 5027231046 |
| Sebastian Elroi Hasian Panjaitan | 5027251040 |
| Kaisar Hanif Pratama | 5027241029 |

Mata kuliah: Keamanan Jaringan Komputer  
Topik: DNS Spoofing, Man-on-the-Side DNS Packet Injection

## Tujuan

Menganalisis traffic pada file PCAP untuk mencari indikasi serangan DNS spoofing. Analisis berfokus pada query DNS yang menerima dua respons dengan DNS Transaction ID sama, tetapi data jawabannya berbeda.

## File dan sumber

- File yang dianalisis: `mycap.pcap`
- Sumber: [waytoalpit/ManOnTheSideAttack-DNS-Spoofing](https://github.com/waytoalpit/ManOnTheSideAttack-DNS-Spoofing)
- File sumber: [`mycap.pcap`](https://github.com/waytoalpit/ManOnTheSideAttack-DNS-Spoofing/blob/master/mycap.pcap)
- Referensi skenario: [`README.txt`](https://github.com/waytoalpit/ManOnTheSideAttack-DNS-Spoofing/blob/master/README.txt)

Repositori sumber menjelaskan bahwa PCAP ini dipakai untuk menguji deteksi DNS poisoning. Pada laporan ini, istilah yang digunakan adalah DNS spoofing atau DNS response injection karena bukti utamanya adalah respons DNS yang dipalsukan.

## Ringkasan traffic

File `mycap.pcap` berisi 78 frame (data size 8.060 byte; ukuran file 9.332 byte). Traffic di dalamnya tidak hanya DNS, tetapi juga ARP, ICMP, TCP, TLS, dan NBNS. Fokus analisis berada pada traffic DNS.

Wireshark menampilkan sepuluh DNS query, antara lain untuk `www.qq.com`, `www.facebook.com`, `www.google.com`, `www.sita.com`, `www.flipkart.com`, `www.yamaha.com`, dan `www.yahoo.com`.

## Langkah analisis di Wireshark

1. Buka `mycap.pcap` di Wireshark.
2. Masukkan filter berikut untuk melihat seluruh DNS traffic.

   ```text
   dns
   ```

3. Untuk melihat query DNS saja, gunakan filter berikut.

   ```text
   dns.flags.response == 0
   ```

4. Fokuskan analisis pada query `www.qq.com` dengan DNS Transaction ID `0x00e5`.

   ```text
   dns.id == 0x00e5
   ```

5. Bandingkan paket 5 dan paket 6. Keduanya adalah respons terhadap query yang sama, tetapi IP jawaban, MAC sumber, dan waktu kedatangannya berbeda.
6. Klik setiap paket dan buka bagian `Ethernet II`, `Internet Protocol Version 4`, `User Datagram Protocol`, dan `Domain Name System` pada panel Packet Details.

## Bukti utama

### Query dan dua respons untuk `www.qq.com`

| Frame | Waktu relatif | Sumber IP | Tujuan IP | DNS ID | Jenis | Nama domain | Jawaban A |
|---:|---:|---|---|---|---|---|---|
| 2 | 1,959863 s | 192.168.88.135 | 8.8.8.8 | `0x00e5` | Query | `www.qq.com` | - |
| 5 | 1,979588 s | 8.8.8.8 | 192.168.88.135 | `0x00e5` | Respons | `www.qq.com` | `192.168.88.134` |
| 6 | 2,217799 s | 8.8.8.8 | 192.168.88.135 | `0x00e5` | Respons | `www.qq.com` | `129.49.1.70`, `129.49.1.73` |

Paket 5 muncul 238,211 ms sebelum paket 6. Keduanya menyatakan diri sebagai respons dari `8.8.8.8` untuk query dan Transaction ID yang sama, tetapi alamat IP pada record A berbeda.

Perbedaan pada header Ethernet dan IPv4 memperkuat indikasi pemalsuan:

| Paket | MAC sumber | IP sumber | TTL | Jawaban DNS |
|---:|---|---|---:|---|
| 5 | `00:0c:29:82:7c:24` | `8.8.8.8` | 64 | `192.168.88.134` |
| 6 | `00:50:56:f5:0b:f8` | `8.8.8.8` | 128 | `129.49.1.70`, `129.49.1.73` |

Paket 5 memakai IP sumber yang sama dengan resolver pada paket 6, tetapi MAC sumbernya berbeda. Jawaban `192.168.88.134` juga merupakan alamat privat, sehingga tidak wajar sebagai alamat publik untuk `www.qq.com`. Pola ini konsisten dengan penyerang yang mengirim respons DNS palsu lebih dahulu sambil memalsukan IP sumber resolver.

### Bukti pendukung: ICMP Port unreachable

Saat filter `dns.id == 0x00e5` diterapkan, selain paket 2, 5, dan 6 akan muncul juga **paket 7 dan 10** bertipe **ICMP "Destination unreachable (Port unreachable)"**. Ini bukan gangguan, melainkan bukti tambahan yang memperkuat analisis: korban sudah menerima respons palsu (paket 5) lebih dahulu, lalu menutup socket UDP untuk query tersebut. Ketika respons asli (paket 6) tiba, port tujuan sudah tertutup, sehingga korban membalas dengan ICMP Port unreachable. Artinya, respons palsu memang tiba dan diproses lebih dulu oleh korban.

## Screenshot yang wajib diambil

Simpan screenshot dengan nama yang sesuai agar mudah dimasukkan ke PPT dan laporan.

| Nama file screenshot | Tampilan yang harus terlihat | Filter Wireshark | Keterangan untuk caption |
|---|---|---|---|
| `01_dns_overview.png` | Packet List berisi traffic DNS dan kolom No., Time, Source, Destination, Protocol, Info | `dns` | Traffic DNS dalam file PCAP yang dianalisis. |
| `02_query_response_qq.png` | Query `www.qq.com` dan respons-responsnya; filter menampilkan paket 2, 5, 6, serta paket 7 dan 10 (ICMP Port unreachable) | `dns.id == 0x00e5` | Satu query `www.qq.com` menerima dua respons DNS dengan ID sama tetapi jawaban IP berbeda; dua paket ICMP menyertai sebagai bukti pendukung. |
| `03_forged_response_frame5.png` | Paket 5 terpilih, panel Packet Details terbuka pada Ethernet II, IPv4, UDP, dan DNS | `frame.number == 5` | Respons yang dicurigai palsu mengarah ke `192.168.88.134` (MAC `00:0c:29:82:7c:24`, TTL 64). |
| `04_legitimate_response_frame6.png` | Paket 6 terpilih, panel Packet Details terbuka pada Ethernet II, IPv4, UDP, dan DNS Answer | `frame.number == 6` | Respons pembanding untuk query yang sama memiliki MAC, TTL, dan IP jawaban berbeda. |
| `05_repeated_pattern_google.png` | Paket query dan respons untuk `www.google.com` | `dns.id == 0xe38c` | Pola respons DNS yang bertentangan juga muncul pada domain lain. |

Saat mengambil screenshot nomor 3 dan 4, pastikan field berikut terlihat:

- DNS Transaction ID: `0x00e5`
- Query name: `www.qq.com`
- A record jawaban
- Source MAC address
- Source IP address
- Time-to-live pada IPv4

Jangan hanya mengambil gambar Packet List tanpa panel Packet Details. Detail header Ethernet, IPv4, dan DNS adalah bukti yang menjelaskan mengapa respons paket 5 dicurigai palsu.

> Catatan status: screenshot `01`, `02`, dan `03` sudah tersedia di folder `screenshots`. Screenshot `04` (paket 6) dan `05` (pola `www.google.com`) masih perlu diambil.

## Hasil identifikasi serangan

Jenis serangan yang ditemukan adalah DNS spoofing atau DNS response injection dengan pola man-on-the-side. Penyerang mengamati query DNS dari korban, lalu mengirim respons palsu yang meniru IP resolver. Respons palsu berusaha mengarahkan korban ke alamat IP yang telah ditentukan penyerang.

Bukti pendukungnya adalah sebagai berikut:

- Satu query DNS untuk `www.qq.com` dengan ID `0x00e5` menerima dua respons.
- Kedua respons memakai IP sumber `8.8.8.8`, tetapi MAC sumber dan nilai TTL berbeda.
- Respons paket 5 datang lebih awal dan memberikan jawaban `192.168.88.134`.
- Respons paket 6 untuk query yang sama memberikan jawaban berbeda, yaitu `129.49.1.70` dan `129.49.1.73`.
- Setelah respons palsu diterima, korban membalas ICMP Port unreachable terhadap respons asli (paket 7 dan 10), menandakan respons palsu tiba lebih dulu.
- Pola respons berbeda dengan DNS ID yang sama juga muncul pada beberapa domain lain (`www.google.com`, `www.sita.com`, `www.flipkart.com`, `www.yamaha.com`).

## Kesimpulan

Analisis terhadap `mycap.pcap` menunjukkan indikasi kuat DNS spoofing. Pada query `www.qq.com`, sebuah respons DNS yang mengarah ke `192.168.88.134` tiba lebih dahulu daripada respons lain dengan Transaction ID yang sama. Perbedaan MAC sumber, TTL, dan A record menunjukkan bahwa respons pertama tidak berasal dari jalur yang sama dengan respons pembanding, walaupun keduanya memakai IP sumber `8.8.8.8`.

Capture ini mendukung kesimpulan adanya upaya pemalsuan respons DNS. Capture saja tidak membuktikan bahwa korban menyimpan jawaban palsu secara permanen di cache resolver atau benar-benar mengakses tujuan palsu. Karena itu, laporan ini menyimpulkan DNS spoofing atau packet injection, bukan menyatakan keberhasilan cache poisoning secara pasti.

## Pembagian bagian PPT

| Slide | Isi |
|---:|---|
| 1 | Judul, anggota kelompok, topik DNS Spoofing. |
| 2 | Tujuan, sumber PCAP, dan alasan memilih `mycap.pcap`. |
| 3 | Ringkasan traffic dan langkah filter `dns`. Gunakan screenshot 1. |
| 4 | Bukti utama query serta dua respons `www.qq.com`. Gunakan screenshot 2. |
| 5 | Detail respons palsu pada paket 5. Gunakan screenshot 3. |
| 6 | Perbandingan dengan paket 6 dan pola tambahan. Gunakan screenshot 4 dan 5. |
| 7 | Jenis serangan, kesimpulan, dan batasan analisis. |
