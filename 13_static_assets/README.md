# Mengelola File Statis (CSS, JS, dan Gambar)

## Deskripsi

Pada bagian ini, kita akan belajar bagaimana cara membuat Pyramid melayani file statis seperti **CSS**, **JavaScript**, dan **gambar** dari direktori tertentu.
File-file ini membantu mempercantik tampilan dan menambah fungsi pada halaman web.

## Tujuan

* Menyediakan direktori yang berisi file statis agar bisa diakses melalui URL.
* Menggunakan fitur Pyramid untuk membuat URL otomatis menuju file-file statis tersebut.

## Langkah-langkah

### 1. Menyalin Proyek Sebelumnya

Salin proyek `view_classes` ke proyek baru bernama `static_assets`:

```bash
cd ..; cp -r view_classes static_assets; cd static_assets
$VENV/bin/pip install -e .
```

Langkah ini membuat salinan proyek sebelumnya agar kita bisa menambahkan fitur file statis tanpa mengubah proyek lama.

---

### 2. Menambahkan Static View pada Konfigurasi

Tambahkan satu baris konfigurasi di file `static_assets/tutorial/__init__.py`:

```python
from pyramid.config import Configurator


def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.include('pyramid_chameleon')
    config.add_route('home', '/')
    config.add_route('hello', '/howdy')
    config.add_static_view(name='static', path='tutorial:static')  # Tambahan baru
    config.scan('.views')
    return config.make_wsgi_app()
```

Bagian `config.add_static_view` berfungsi agar Pyramid bisa melayani file statis dari folder `static` di dalam package `tutorial`.

---

### 3. Menambahkan Link CSS pada Template

Edit file `static_assets/tutorial/home.pt` dan tambahkan tag `<link>` di bagian `<head>` agar file CSS bisa digunakan:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Quick Tutorial: ${name}</title>
    <link rel="stylesheet"
          href="${request.static_url('tutorial:static/app.css') }"/>
</head>
<body>
<h1>Hi ${name}</h1>
</body>
</html>
```

Fungsi `request.static_url()` digunakan untuk membuat URL otomatis ke file CSS sesuai konfigurasi aplikasi.

---

### 4. Membuat File CSS

Buat folder `static` di dalam `tutorial`, lalu tambahkan file `app.css`:

```css
body {
    margin: 2em;
    font-family: sans-serif;
}
```

File ini akan mengatur tampilan teks di halaman web agar lebih rapi dan mudah dibaca.

---

### 5. Menambahkan Functional Test

Tambahkan pengujian berikut untuk memastikan file CSS bisa diakses dengan benar:

```python
def test_css(self):
    res = self.testapp.get('/static/app.css', status=200)
    self.assertIn(b'body', res.body)
```

Test ini memastikan server benar-benar dapat memberikan file `app.css` saat diminta melalui URL `/static/app.css`.

---

### 6. Menjalankan Test

Jalankan pengujian menggunakan perintah berikut:

```bash
$VENV/bin/pytest tutorial/tests.py -q
```

Jika berhasil, hasil test akan menampilkan pesan seperti berikut:

```
....
5 passed in 0.50 seconds
```

Hasil:
<img width="1241" height="445" alt="Screenshot 2025-11-14 000142" src="https://github.com/user-attachments/assets/ff8978f6-f5ad-4c02-8486-c360ca1085c7" />

---

### 7. Menjalankan Aplikasi

Gunakan perintah berikut untuk menjalankan aplikasi Pyramid:

```bash
$VENV/bin/pserve development.ini --reload
```

Buka **[http://localhost:6543/](http://localhost:6543/)** di browser.
Jika CSS berhasil dimuat, kamu akan melihat perubahan tampilan font pada halaman web.

Hasil:
<img width="1919" height="1021" alt="Screenshot 2025-11-14 000204" src="https://github.com/user-attachments/assets/31c5cfb5-540a-4a07-8fb3-b10942e4737f" />

---

## Analisis

### Pemetaaan File Statis

Konfigurasi `config.add_static_view` membuat Pyramid memetakan permintaan ke `http://localhost:6543/static/` agar diarahkan ke folder `tutorial/static`.
Folder ini berisi file `app.css` yang digunakan di template.

### Menggunakan Helper Pyramid

Di template, kita bisa saja menulis:

```html
<link rel="stylesheet" href="/static/app.css">
```

Namun cara ini tidak fleksibel. Jika struktur direktori berubah atau situs dipindahkan ke lokasi lain, link bisa menjadi salah.
Dengan menggunakan:

```python
${request.static_url('tutorial:static/app.css')}
```

URL akan dibuat otomatis sesuai konfigurasi yang sudah ditetapkan, sehingga aman dari perubahan struktur.

---

## Perbandingan Fungsi URL

| Fungsi                  | Hasil                                                                    | Kegunaan                                            |
| ----------------------- | ------------------------------------------------------------------------ | --------------------------------------------------- |
| `request.static_url()`  | Menghasilkan URL lengkap, contoh: `http://localhost:6543/static/app.css` | Cocok untuk link eksternal atau API                 |
| `request.static_path()` | Menghasilkan path relatif, contoh: `/static/app.css`                     | Cocok untuk penggunaan internal dalam satu aplikasi |

Contoh:

```python
${request.static_url('tutorial:static/app.css')}
# Hasil: http://localhost:6543/static/app.css

${request.static_path('tutorial:static/app.css')}
# Hasil: /static/app.css
```

Gunakan `request.static_path()` jika situs hanya berjalan secara lokal, dan `request.static_url()` bila perlu URL lengkap.

---

## Kesimpulan

Dalam tutorial ini, kita telah mempelajari cara:

1. Menambahkan konfigurasi `config.add_static_view` untuk melayani file statis.
2. Membuat folder khusus untuk CSS, JS, dan gambar.
3. Menghubungkan file CSS ke template dengan `request.static_url()`.
4. Menjalankan pengujian untuk memastikan file statis berfungsi.
5. Memahami perbedaan antara `static_url` dan `static_path`.

Dengan langkah-langkah ini, aplikasi Pyramid dapat menampilkan file statis dengan cara yang **fleksibel, efisien, dan mudah diatur**.
