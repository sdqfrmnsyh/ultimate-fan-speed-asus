# Ultimate Fan Speed ASUS

Systemd service sederhana untuk mengatur mode kontrol kipas laptop ASUS saat proses boot selesai.

Service ini menjalankan perintah berikut:

- Menunggu selama 2 detik setelah service dimulai.
- Menemukan semua file `pwm1_enable` di `/sys/devices/platform/asus-nb-wmi/hwmon/hwmon*/`.
- Menulis nilai `0` ke setiap file tersebut.
- Tetap berstatus aktif setelah perintah selesai melalui `RemainAfterExit=yes`.

Pada banyak perangkat ASUS, nilai `0` pada `pwm1_enable` mengembalikan kontrol kipas ke mode manual. Perilaku ini bergantung pada dukungan kernel dan perangkat yang digunakan.

## Prasyarat

- Linux dengan `systemd`.
- Kernel yang memuat driver ASUS WMI (`asus-nb-wmi`).
- File `/sys/devices/platform/asus-nb-wmi/hwmon/hwmon*/pwm1_enable` tersedia.
- Akses `root` atau akun dengan izin `sudo`.

## Instalasi

Salin unit service ke direktori systemd lokal:

```bash
sudo install -m 644 asus-fan-init.service /etc/systemd/system/asus-fan-init.service
```

Muat ulang konfigurasi systemd, lalu aktifkan service agar berjalan setiap boot:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now asus-fan-init.service
```

`--now` menjalankan service segera setelah diaktifkan. Jika hanya ingin mengaktifkannya untuk boot berikutnya, gunakan:

```bash
sudo systemctl enable asus-fan-init.service
```

## Verifikasi

Periksa status service:

```bash
systemctl status asus-fan-init.service
```

Service yang berhasil dijalankan akan menampilkan status `active (exited)`. Periksa apakah file kontrol kipas tersedia dan nilainya sudah berubah:

```bash
for f in /sys/devices/platform/asus-nb-wmi/hwmon/hwmon*/pwm1_enable; do
    printf '%s: ' "$f"
    cat "$f"
done
```

Log boot service dapat dilihat dengan:

```bash
journalctl -u asus-fan-init.service
```

## Menonaktifkan atau menghapus

Untuk menghentikan service agar tidak dijalankan pada boot berikutnya:

```bash
sudo systemctl disable --now asus-fan-init.service
```

Untuk menghapus unit yang sudah dipasang:

```bash
sudo rm /etc/systemd/system/asus-fan-init.service
sudo systemctl daemon-reload
```

## Troubleshooting

### Service berhasil tetapi kipas tidak berubah

Pastikan path kontrol tersedia:

```bash
ls /sys/devices/platform/asus-nb-wmi/hwmon/hwmon*/pwm1_enable
```

Jika tidak ada file yang cocok, driver ASUS WMI mungkin belum aktif, nama path hwmon berbeda, atau perangkat tidak menyediakan kontrol PWM tersebut.

### Service gagal saat boot

Lihat pesan error lengkap:

```bash
systemctl status asus-fan-init.service --no-pager
journalctl -u asus-fan-init.service -b --no-pager
```

### Peringatan perangkat keras

Kontrol kipas secara manual dapat menyebabkan suhu meningkat jika kurva atau kecepatan kipas tidak diatur dengan benar. Pantau suhu perangkat dan pastikan konfigurasi kipas sesuai dengan model ASUS yang digunakan.

## Lisensi

Belum ada lisensi yang ditentukan untuk proyek ini.
