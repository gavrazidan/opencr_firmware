# Troubleshooting IMU ROBOTIS OP3 – ROS 2

## Masalah

Saat menjalankan **bringup ROBOTIS OP3**, ROS 2 berhasil menampilkan topic IMU, tetapi seluruh data IMU bernilai `0`.

Contoh:

```text
/robotis/open_cr/imu
````

Data yang muncul:

```text
orientation:
  x: 0.0
  y: 0.0
  z: 0.0
  w: 0.0

angular_velocity:
  x: 0.0
  y: 0.0
  z: 0.0

linear_acceleration:
  x: 0.0
  y: 0.0
  z: 0.0
```

Masalah ini menyebabkan **yaw tidak terbaca** dan dapat menyebabkan proses bringup OP3 mengalami error.

---

## 1. Alur Data IMU pada OP3

```text
IMU Sensor
    │
    ▼
  OpenCR
    │
    ▼
USB / Serial
    │
    ▼
open_cr_module
    │
    ▼
/robotis/open_cr/imu
    │
    ▼
ROS 2 / OP3
```

Jika topic `/robotis/open_cr/imu` ada tetapi seluruh datanya `0`, kemungkinan masalah berada pada:

* IMU → OpenCR
* Firmware OpenCR
* komunikasi OpenCR → PC
* `open_cr_module`

---

## 2. Cek Topic IMU

Jalankan:

```bash
ros2 topic list | grep open_cr
```

Pastikan terdapat:

```text
/robotis/open_cr/imu
```

Kemudian:

```bash
ros2 topic echo /robotis/open_cr/imu
```

Gerakkan atau miringkan robot.

### Kondisi normal

Nilai accelerometer, gyro, dan orientation seharusnya berubah ketika robot digerakkan.

### Kondisi bermasalah

Jika tetap:

```text
x: 0.0
y: 0.0
z: 0.0
```

meskipun robot digerakkan, kemungkinan OpenCR tidak memberikan data IMU yang valid.

---

## 3. Cek Frekuensi Topic

Jalankan:

```bash
ros2 topic hz /robotis/open_cr/imu
```

Jika muncul:

```text
average rate: 100.0
```

berarti ROS 2 menerima message secara periodik.

Namun jika message masuk dengan frekuensi normal tetapi semua nilai `0`, masalah kemungkinan bukan pada publisher ROS 2, melainkan data yang dikirim dari OpenCR.

---

## 4. Periksa Firmware OpenCR

OP3 menggunakan firmware OpenCR khusus OP3.

Pada Arduino IDE:

```text
File
 └── Examples
      └── OP3
           └── opencr_op3
```

Contoh tersebut merupakan firmware yang digunakan untuk fungsi OpenCR pada OP3.

### Penting

Jangan langsung menggunakan contoh IMU biasa seperti:

```text
Examples
 └── Sensors
      └── IMU
```

untuk menggantikan firmware OP3.

Firmware OP3 menangani fungsi yang dibutuhkan robot, termasuk komunikasi dengan sistem OP3.

---

## 5. Kemungkinan Firmware Tidak Sesuai

Jika sebelumnya OpenCR pernah:

* di-upload program Arduino lain
* di-upload contoh sensor
* diubah program `opencr_op3`
* gagal upload firmware
* menggunakan firmware OpenCR untuk board/project lain

maka firmware OpenCR perlu dicurigai.

Gejalanya:

```text
Bringup
   │
   ├── OpenCR terdeteksi
   ├── Topic IMU muncul
   └── Data IMU = 0
```

---

## 6. Jangan Langsung Kalibrasi Yaw

Kalibrasi IMU bukan langkah pertama jika:

```text
gyro = 0
accelerometer = 0
orientation = 0
yaw = 0
```

Kalibrasi lebih tepat dilakukan jika sensor sudah memberikan data tetapi offset atau orientasinya salah.

Jika semua data sensor nol, periksa terlebih dahulu:

1. Firmware OpenCR
2. koneksi OpenCR
3. komunikasi serial
4. inisialisasi IMU
5. `open_cr_module`

---

## 7. Checklist Pemeriksaan

### OpenCR

* [ ] OpenCR menyala
* [ ] Kabel USB/serial terhubung dengan benar
* [ ] Firmware OP3 terpasang
* [ ] Tidak ada firmware Arduino lain yang menggantikan firmware OP3

### ROS 2

* [ ] `/robotis/open_cr/imu` tersedia
* [ ] Topic memiliki publisher
* [ ] Topic memiliki frekuensi
* [ ] Data IMU tidak semuanya `0`

### IMU

* [ ] Accelerometer berubah saat robot dimiringkan
* [ ] Gyroscope berubah saat robot diputar
* [ ] Orientation berubah
* [ ] Yaw berubah saat robot diputar

---

## 8. Perintah Diagnostik

### Melihat topic

```bash
ros2 topic list
```

### Filter OpenCR

```bash
ros2 topic list | grep open_cr
```

### Melihat data IMU

```bash
ros2 topic echo /robotis/open_cr/imu
```

### Melihat frekuensi IMU

```bash
ros2 topic hz /robotis/open_cr/imu
```

### Melihat informasi topic

```bash
ros2 topic info /robotis/open_cr/imu
```

### Melihat node

```bash
ros2 node list
```

Cari node yang berkaitan dengan:

```text
open_cr
op3
imu
```

---

## 9. Kesimpulan Sementara

Jika kondisi robot:

```text
OP3 Bringup
      │
      ▼
Topic /robotis/open_cr/imu tersedia
      │
      ▼
Message terus masuk
      │
      ▼
Semua data IMU = 0
      │
      ▼
Yaw tidak terbaca
```

maka fokus pemeriksaan utama adalah **OpenCR dan firmware OpenCR**, bukan algoritma yaw ROS 2.

Urutan troubleshooting:

```text
1. Cek /robotis/open_cr/imu
          ↓
2. Cek ros2 topic hz
          ↓
3. Cek ros2 topic info
          ↓
4. Cek koneksi OpenCR
          ↓
5. Cek firmware opencr_op3
          ↓
6. Jika perlu, restore firmware OP3
          ↓
7. Jalankan bringup kembali
          ↓
8. Cek IMU dan yaw
```

---

## Catatan

Sebelum melakukan flashing firmware, simpan terlebih dahulu kondisi firmware/program OpenCR saat ini jika memungkinkan.

Jika setelah restore firmware OP3 data IMU masih `0`, pemeriksaan berikutnya harus difokuskan pada **hardware IMU/OpenCR dan komunikasi serial**, bukan hanya software ROS 2.

```
```

