# Templating dengan Jinja2

## Deskripsi

Tutorial ini menjelaskan cara menggunakan **Jinja2** sebagai sistem template di framework **Pyramid**.
Jinja2 adalah salah satu template engine populer di Python, juga digunakan oleh **Flask** dan memiliki konsep serupa dengan template di **Django**.
Melalui tutorial ini, kita akan melihat bahwa Pyramid mendukung berbagai jenis sistem templating.

## Tujuan Tutorial

* Menunjukkan bahwa Pyramid dapat menggunakan lebih dari satu jenis sistem template.
* Mempelajari cara menambahkan dan mengaktifkan **add-on (plugin)** di Pyramid.

---

## Langkah-langkah Pengerjaan

### 1. Menyalin Proyek Sebelumnya

Kita akan mulai dari proyek sebelumnya (`view_classes`).
Salin folder tersebut sebagai dasar proyek baru:

```bash
cd ..; cp -r view_classes jinja2; cd jinja2
```

---

### 2. Menambahkan Dependency di `setup.py`

Tambahkan package **pyramid_jinja2** ke daftar dependency di file `setup.py`:

```python
requires = [
    'pyramid',
    'pyramid_chameleon',
    'pyramid_jinja2',  # Tambahan baru
    'waitress',
]
```

`setup.py` berfungsi untuk mendefinisikan semua library yang diperlukan oleh proyek agar dapat berjalan dengan benar.

---

### 3. Menginstal Dependency

Setelah menambahkan dependensi, jalankan perintah berikut untuk menginstal semua kebutuhan proyek, termasuk Jinja2:

```bash
$VENV/bin/pip install -e .
```

---

### 4. Mengaktifkan Jinja2 di Konfigurasi Proyek

Buka file `tutorial/__init__.py` dan tambahkan baris berikut agar Pyramid mengenali Jinja2:

```python
def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.include('pyramid_jinja2')  # Mengaktifkan Jinja2
    config.add_route('home', '/')
    config.add_route('hello', '/howdy')
    config.scan('.views')
    return config.make_wsgi_app()
```

Perintah `config.include('pyramid_jinja2')` memberi tahu Pyramid bahwa kita akan menggunakan Jinja2 sebagai sistem template.

---

### 5. Mengubah Renderer di File Views

Buka file `tutorial/views.py`, lalu ubah bagian renderer dari `.pt` menjadi `.jinja2`:

```python
from pyramid.view import (
    view_config,
    view_defaults
    )

@view_defaults(renderer='home.jinja2')  # Ganti ekstensi template
class TutorialViews:
    def __init__(self, request):
        self.request = request

    @view_config(route_name='home')
    def home(self):
        return {'name': 'Home View'}

    @view_config(route_name='hello')
    def hello(self):
        return {'name': 'Hello View'}
```

Pada bagian ini, kita hanya mengubah nama file template yang digunakan, tanpa mengubah logika utama.

---

### 6. Membuat Template Jinja2

Selanjutnya, buat file baru bernama `tutorial/home.jinja2`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Quick Tutorial: {{ name }}</title>
</head>
<body>
    <h1>Hi {{ name }}</h1>
</body>
</html>
```

Di dalam Jinja2, variabel ditulis menggunakan tanda kurung ganda `{{ ... }}`.
Misalnya, `{{ name }}` akan diganti dengan nilai yang dikirim dari view.

---

### 7. Menjalankan Test

Pastikan semua pengujian tetap berjalan dengan baik:

```bash
$VENV/bin/pytest tutorial/tests.py -q
```

Jika berhasil, akan muncul hasil seperti:

```
....
4 passed in 0.40 seconds
```

Hasil:
<img width="1238" height="444" alt="Screenshot 2025-11-13 234154" src="https://github.com/user-attachments/assets/6964cf0b-b0fd-461b-884b-77ddb9c93134" />

---

### 8. Menjalankan Aplikasi

Sekarang jalankan server Pyramid:

```bash
$VENV/bin/pserve development.ini --reload
```

---

### 9. Melihat Hasil di Browser

Buka alamat berikut di browser Anda:

```
http://localhost:6543/
```

Tampilan halaman akan menampilkan teks seperti:

```
Hi Home View
```

Hasil:
<img width="1919" height="1020" alt="Screenshot 2025-11-13 234215" src="https://github.com/user-attachments/assets/2a1b0468-7062-41ea-957e-80dac1fe2931" />

---

## Analisis: Bagaimana Prosesnya Bekerja?

### Cara Memasang Add-on di Pyramid

Menambahkan add-on di Pyramid sangat mudah:

1. **Install add-on** menggunakan pip ke dalam virtual environment.
2. **Aktifkan** add-on tersebut di dalam konfigurasi menggunakan `config.include()`.

Ketika Pyramid menjalankan baris `config.include('pyramid_jinja2')`, sistem akan menyiapkan renderer baru untuk file dengan ekstensi `.jinja2`.
Dengan begitu, Pyramid tahu bagaimana cara menampilkan file template jenis ini.

---

### Perubahan Kecil pada Kode

Hal menarik dari proses ini adalah:
kita tidak perlu banyak mengubah kode lama — cukup ganti ekstensi file template dari `.pt` menjadi `.jinja2`.
Sintaks dasar untuk menampilkan variabel juga hampir sama, sehingga peralihan dari Chameleon ke Jinja2 terasa sangat mudah.

Hal ini menunjukkan bahwa Pyramid sangat **fleksibel** dan dapat bekerja dengan berbagai sistem templating tanpa harus mengubah struktur aplikasi secara besar-besaran.

---

## Pertanyaan Tambahan (Extra Credit)

### 1. Cara Lain Menambahkan Dependency

**Pertanyaan:**
Sekarang proyek menggunakan `pyramid_jinja2` dan sudah diinstal secara manual. Apakah ada cara lain untuk menautkannya?

**Jawaban:**
Ya, dengan menambahkan `pyramid_jinja2` ke daftar `install_requires` di `setup.py`, maka setiap kali proyek diinstal menggunakan `pip install -e .`, package tersebut otomatis ikut terpasang.
Alternatif lain:

* Gunakan file `requirements.txt` dan jalankan `pip install -r requirements.txt`.
* Gunakan manajer dependensi seperti **Poetry** atau **Pipenv** untuk otomatisasi instalasi.

---

### 2. Cara Lain Mengaktifkan Add-on

**Pertanyaan:**
Selain `config.include('pyramid_jinja2')`, apakah ada cara lain untuk memuat add-on ke dalam aplikasi?

**Jawaban:**
Ada. Kita bisa menggunakan pendekatan **deklaratif** melalui file konfigurasi `.ini`.
Tambahkan baris berikut di `development.ini`:

```ini
[app:main]
pyramid.includes = pyramid_jinja2
```

Dengan cara ini, Pyramid akan otomatis memuat add-on saat aplikasi dijalankan, tanpa perlu menulis baris `config.include()` di kode Python.

Keuntungan metode ini:

* Pengaturan konfigurasi terpisah dari kode program.
* Mudah mengubah konfigurasi untuk mode development atau production tanpa menyentuh kode utama.

---

## Kesimpulan

Dalam tutorial ini, kita telah belajar bahwa:

* Pyramid mendukung berbagai sistem template seperti Chameleon dan Jinja2.
* Menambahkan add-on di Pyramid sangat sederhana: cukup instal package dan aktifkan di konfigurasi.
* Hanya dengan mengganti ekstensi file renderer, kita bisa langsung menggunakan template engine lain tanpa mengubah struktur aplikasi.

Pendekatan ini menunjukkan bahwa Pyramid dirancang agar **mudah diadaptasi, fleksibel, dan efisien**, bahkan untuk pemula yang baru belajar framework web di Python.
