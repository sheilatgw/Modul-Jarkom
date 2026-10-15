# Tutorial Praktikum Modul 1 — Topologi Jaringan Dasar

> **Catatan:** Ganti `[IP Prefix]` sesuai pembagian IP yang diberikan. Contoh `192.168` menghasilkan alamat `192.168.100.101`.
>
> **Prerequisite:** GNS3 VM dan GNS3-GUI sudah terpasang, appliance `netics-pc` tersedia, dan GNS3 console dapat diakses.
>
> **Dokumentasi wajib:** Tugas Mandiri Bagian A dan Tugas Mandiri Bagian B. Pada setiap bagian laporan yang diminta, sertakan screenshot, penjelasan singkat, dan kode/perintah yang digunakan.

---

## A. Point-to-Point

### A.1 Menghubungkan Dua PC

1. Di GNS3, tambahkan dua node `netics-pc`, lalu beri nama `netics-pc-1` dan `netics-pc-2`.
2. Hubungkan port `eth0` pada kedua PC.

   **INPUT SCREENSHOT —** Topologi GNS3 yang menunjukkan `netics-pc-1` terhubung langsung ke `netics-pc-2`.
3. Start kedua node dan buka console masing-masing.
4. Konfigurasikan IP address.

   Pada `netics-pc-1`:
   ```bash
   ip addr add [IP Prefix].100.101/24 dev eth0
   ip link set eth0 up
   ```

   Pada `netics-pc-2`:
   ```bash
   ip addr add [IP Prefix].100.102/24 dev eth0
   ip link set eth0 up
   ```
5. Verifikasi IP pada kedua PC:
   ```bash
   ip -br addr
   ```
   **INPUT SCREENSHOT —** Output `ip -br addr` pada `netics-pc-1` dan `netics-pc-2`.
6. Uji konektivitas dari masing-masing PC:
   ```bash
   ping [IP Prefix].100.102
   ```
   Jalankan dari `netics-pc-1`, lalu ping balik ke `[IP Prefix].100.101` dari `netics-pc-2`.

   **INPUT SCREENSHOT —** Hasil ping yang menunjukkan paket mendapat reply.

### A.2 Mengukur Kapasitas dan Throughput Link

1. Jalankan perintah berikut di kedua PC untuk memeriksa kapasitas interface:
   ```bash
   ethtool eth0
   ```
   **INPUT SCREENSHOT —** Output `ethtool eth0` yang memperlihatkan informasi `Speed`.
2. Di `netics-pc-1`, jalankan iperf3 sebagai server:
   ```bash
   iperf3 -s
   ```
   **INPUT SCREENSHOT —** Console `netics-pc-1` yang menunjukkan iperf3 sedang menunggu koneksi.
3. Di `netics-pc-2`, jalankan pengujian UDP dengan target bitrate 10 Gbps:
   ```bash
   iperf3 -c [IP Prefix].100.101 -u -b 10G
   ```
   Ganti alamat target sesuai IP `netics-pc-1`.

   **INPUT SCREENSHOT —** Output iperf3 di sisi client yang memperlihatkan hasil pengukuran sender dan receiver.

### A.3 Ethernet Bridge dan Simulasi Gangguan

#### Membuat bridge

1. Tambahkan node `netics-pc-3` dengan dua interface dan tempatkan di antara `netics-pc-1` dan `netics-pc-2`.
2. Hubungkan:
   - `netics-pc-1 eth0` ke `netics-pc-3 eth0`
   - `netics-pc-3 eth1` ke `netics-pc-2 eth0`
3. Pastikan IP `netics-pc-1` tetap `[IP Prefix].100.101/24` dan `netics-pc-2` tetap `[IP Prefix].100.102/24`.

   **INPUT SCREENSHOT —** Topologi GNS3 dengan `netics-pc-3` berada di antara kedua PC.
4. Sebelum mengaktifkan bridge, jalankan dari `netics-pc-1`:
   ```bash
   mtr [IP Prefix].100.102
   ```
   **INPUT SCREENSHOT —** Hasil mtr sebelum bridge aktif.
5. Di `netics-pc-3`, periksa interface lalu buat dan aktifkan bridge:
   ```bash
   ip -br addr
   brctl addbr br0
   brctl addif br0 eth0
   brctl addif br0 eth1
   brctl show
   ip link set br0 up
   ```
6. Ulangi ping atau mtr dari `netics-pc-1` ke `netics-pc-2`.

   **INPUT SCREENSHOT —** Perintah konfigurasi bridge di `netics-pc-3` dan hasil ping/mtr yang berhasil setelah bridge aktif.

#### Simulasi packet loss 10%

1. Di `netics-pc-3`, terapkan packet loss pada kedua interface:
   ```bash
   tc qdisc replace dev eth1 root netem loss 10%
   tc qdisc replace dev eth0 root netem loss 10%
   ```
2. Jalankan mtr dari `netics-pc-1` ke `netics-pc-2` untuk mengamati packet loss.

   **INPUT SCREENSHOT —** Perintah `tc` dan hasil mtr setelah packet loss diterapkan.

#### Membatasi throughput menjadi 50 Mbps

1. Di `netics-pc-3`, terapkan pembatasan pada kedua interface:
   ```bash
   tc qdisc replace dev eth0 root tbf rate 50mbit burst 64k limit 64k
   tc qdisc replace dev eth1 root tbf rate 50mbit burst 64k limit 64k
   ```
2. Untuk menghapus pembatasan:
   ```bash
   tc qdisc del dev eth0 root
   tc qdisc del dev eth1 root
   ```
3. Untuk menguji throughput, jalankan `iperf3 -s` di `netics-pc-1`, lalu jalankan client dari `netics-pc-2`:
   ```bash
   iperf3 -c [IP Prefix].100.101 -u -b 100M
   ```

   **INPUT SCREENSHOT —** Perintah pembatasan throughput dan hasil iperf3 yang menunjukkan bitrate mendekati 50 Mbps.

### Tugas Mandiri Bagian A — Topologi 3 PC melalui Bridge

1. Buat topologi dengan `netics-pc-1`, `netics-pc-2`, dan `netics-pc-4` yang terhubung melalui `netics-pc-3` sebagai bridge. Sesuaikan prefix IP dengan pembagian yang diberikan.

   **INPUT SCREENSHOT —** Topologi buatan sendiri di GNS3 yang memperlihatkan ketiga PC melalui bridge `netics-pc-3`.
2. Konfigurasikan IP address pada setiap PC dan pastikan ketiganya dapat saling berkomunikasi.
3. Buktikan konektivitas antar-PC menggunakan `ping` dan `mtr`.

   **INPUT SCREENSHOT —** Konfigurasi IP dan hasil ping/mtr antar-PC.
4. Terapkan pembatasan throughput 50 Mbps pada interface bridge, lalu buktikan hasilnya.

   **INPUT SCREENSHOT —** Perintah pembatasan throughput dan hasil pengujian.
5. Terapkan packet loss 20% pada interface bridge, lalu buktikan hasilnya.

   **INPUT SCREENSHOT —** Perintah `tc netem` dan hasil mtr yang menunjukkan packet loss.
6. Untuk laporan, sertakan topologi buatan sendiri, konfigurasi IP tiap node, bukti konektivitas, implementasi packet loss, serta implementasi pembatasan throughput. Setiap poin dilengkapi screenshot, penjelasan singkat, dan kode/perintah.

---

## B. Point-to-Multipoint — Hub vs Switch

### B.1 Topologi dengan Ethernet Hub

1. Buat topologi star menggunakan satu Ethernet hub dan empat node: `netics-pc-1` sampai `netics-pc-4`.
2. Hubungkan interface `eth0` setiap PC ke port hub:
   - `netics-pc-1 eth0` → `Ethernet0`
   - `netics-pc-2 eth0` → `Ethernet1`
   - `netics-pc-3 eth0` → `Ethernet2`
   - `netics-pc-4 eth0` → `Ethernet3`

   **INPUT SCREENSHOT —** Topologi star empat PC yang terhubung ke Ethernet hub.
3. Konfigurasikan IP address di `eth0` masing-masing PC:
   | Perangkat | IP address | Netmask |
   |---|---|---|
   | `netics-pc-1` | `[IP Prefix].100.101` | `255.255.255.0 (/24)` |
   | `netics-pc-2` | `[IP Prefix].100.102` | `255.255.255.0 (/24)` |
   | `netics-pc-3` | `[IP Prefix].100.103` | `255.255.255.0 (/24)` |
   | `netics-pc-4` | `[IP Prefix].100.104` | `255.255.255.0 (/24)` |

   Contoh perintah untuk setiap PC (sesuaikan alamat IP):
   ```bash
   ip addr add [IP address]/24 dev eth0
   ip link set eth0 up
   ```
4. Uji konektivitas menggunakan `ping` antar-PC. Pastikan setiap pengujian mendapat reply.

   **INPUT SCREENSHOT —** Hasil ping antar-PC.
5. Tambahkan `netics-pc-5`, hubungkan ke port hub yang masih tersedia, lalu start node.
6. Di `netics-pc-5`, mulai packet capture:
   ```bash
   termshark -i eth0
   ```
7. Dari `netics-pc-1`, ping `netics-pc-2`:
   ```bash
   ping [IP Prefix].100.102
   ```
8. Amati apakah traffic ping terlihat di `termshark` pada `netics-pc-5`.

   **INPUT SCREENSHOT —** Capture `termshark` di `netics-pc-5` yang menunjukkan traffic ping antara `netics-pc-1` dan `netics-pc-2`.

### B.2 Mengganti Hub dengan Ethernet Switch

1. Ganti Ethernet hub pada topologi B.1 dengan Ethernet switch. Pertahankan susunan PC, koneksi, dan konfigurasi IP yang sama.

   **INPUT SCREENSHOT —** Topologi yang sama dengan Ethernet switch sebagai pengganti hub.
2. Jalankan kembali capture di `netics-pc-5`:
   ```bash
   termshark -i eth0
   ```
3. Dari `netics-pc-1`, ping `netics-pc-2` dengan perintah yang sama seperti pada B.1.
4. Amati hasil capture. Traffic ping unicast antara `netics-pc-1` dan `netics-pc-2` diharapkan tidak terlihat di `netics-pc-5`, kecuali traffic broadcast seperti ARP.

   **INPUT SCREENSHOT —** Hasil capture `termshark` pada skenario switch.

### Tugas Mandiri Bagian B — Observasi Hub vs Switch

1. Buat topologi dengan lima PC (`netics-pc-1` sampai `netics-pc-5`) yang terhubung ke satu Ethernet switch. Tambahkan `netics-pc-6` sebagai capture point pada port switch yang tersedia.

   **INPUT SCREENSHOT —** Topologi buatan sendiri sesuai konfigurasi tugas.
2. Konfigurasikan IP address kelima PC dalam subnet yang sama. Buktikan seluruh PC dapat saling ping.

   **INPUT SCREENSHOT —** Konfigurasi IP dan bukti ping antar-PC.
3. Jalankan capture di `netics-pc-6`:
   ```bash
   termshark -i eth0
   ```
4. Buat traffic ping dari `netics-pc-1` ke `netics-pc-2`. Catat apakah traffic tersebut tertangkap di `netics-pc-6`.

   **INPUT SCREENSHOT —** Hasil capture saat menggunakan switch.
5. Ganti switch dengan hub tanpa mengubah konfigurasi IP PC. Ulangi capture dan skenario ping yang sama.

   **INPUT SCREENSHOT —** Hasil capture saat menggunakan hub.
6. Bandingkan hasil capture hub dan switch. Tuliskan kesimpulan berdasarkan observasi mengenai broadcast domain dan collision domain.
7. **Tambahan:** Tambahkan `netics-pc-7` pada port terpisah di switch/hub. Amati apakah traffic broadcast seperti ARP request diterima PC ketujuh pada kedua skenario.

   **INPUT SCREENSHOT —** Bukti observasi traffic broadcast pada skenario switch dan hub.
8. Laporan wajib mencakup topologi buatan sendiri, konfigurasi IP, bukti bahwa traffic ping/mtr tidak tertangkap di capture point saat memakai switch, bukti bahwa traffic tertangkap saat memakai hub, serta kesimpulan berdasarkan hasil observasi. Sertakan screenshot, penjelasan singkat, dan kode/perintah pada setiap poin yang relevan.

---

## C. Konfigurasi IP Otomatis via DHCP

### C.1 DHCP Server dan DHCP Client

1. Buat topologi star menggunakan Ethernet switch. Hubungkan `eth0` dari `netics-pc-1` sampai `netics-pc-4` ke port switch.

   **INPUT SCREENSHOT —** Topologi empat PC dengan switch.
2. Atur konfigurasi jaringan pada `netics-pc-1` sampai `netics-pc-4` agar interface `eth0` menggunakan DHCP. Pada konfigurasi jaringan masing-masing node, gunakan:
   ```text
   auto eth0
   iface eth0 inet dhcp
       hostname netics-pc-xxx
   ```
   Ganti `netics-pc-xxx` sesuai nama node, lalu terapkan konfigurasi.

   **INPUT SCREENSHOT —** Pengaturan network configuration DHCP client di GNS3.
3. Tambahkan `netics-pc-5` sebagai DHCP server dan hubungkan ke port switch yang tersedia.
4. Di `netics-pc-5`, atur IP statis pada `eth0`:
   ```bash
   ip addr add [IP Prefix].200.10/24 dev eth0
   ip link set eth0 up
   ```
5. Buat file untuk menyimpan lease:
   ```bash
   touch /tmp/dhcpd.leases
   ```
6. Buat file konfigurasi DHCP di `/tmp/dhcpd.conf` dengan isi berikut:
   ```text
   start [IP Prefix].200.100
   end [IP Prefix].200.150
   interface eth0
   max_leases 50
   pidfile /tmp/dhcpd.pid
   lease_file /tmp/dhcpd.leases
   option subnet 255.255.255.0
   ```
7. Jalankan DHCP server:
   ```bash
   udhcpd -f /tmp/dhcpd.conf
   ```

   **INPUT SCREENSHOT —** Console `netics-pc-5` yang menunjukkan DHCP server aktif dan melayani request client.
8. Periksa alamat IP pada `netics-pc-1` sampai `netics-pc-4`:
   ```bash
   ip -br addr
   ```
   Pastikan masing-masing mendapatkan IP dari pool DHCP `[IP Prefix].200.100` sampai `[IP Prefix].200.150`.
9. Uji konektivitas antar-PC dengan `ping`.

   **INPUT SCREENSHOT —** Output `ip -br addr` yang memperlihatkan alamat hasil DHCP dan hasil ping antar-PC.

### DHCP Client yang Dijalankan Manual

1. Pada `netics-pc-6`, periksa alamat IP saat ini:
   ```bash
   ip -br addr
   ```
2. Jalankan DHCP client secara manual pada interface `eth0` di background:
   ```bash
   udhcpc -i eth0 -b
   ```
3. Periksa kembali alamat IP:
   ```bash
   ip -br addr
   ```

   **INPUT SCREENSHOT —** Perbandingan kondisi sebelum dan sesudah menjalankan `udhcpc`, yang menunjukkan `netics-pc-6` memperoleh IP.

### Observasi Aktivitas DHCP melalui Capture

1. Tambahkan `netics-pc-7` sebagai capture point pada topologi dan start node.
2. Di `netics-pc-7`, jalankan:
   ```bash
   termshark -i eth0
   ```

   **INPUT SCREENSHOT —** `termshark` berjalan dan menunggu traffic pada interface `eth0`.
3. Restart `netics-pc-4` agar DHCP client meminta konfigurasi IP kembali.
4. Amati paket DHCP yang tertangkap, terutama rangkaian Discover, Offer, dan Request.

   **INPUT SCREENSHOT —** Capture `termshark` yang memperlihatkan paket DHCP dari proses permintaan alamat IP.
