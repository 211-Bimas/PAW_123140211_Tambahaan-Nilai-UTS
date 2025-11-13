# Pengembangan AJAX dengan JSON Renderer

## Deskripsi

Aplikasi web modern tidak hanya menampilkan halaman HTML. Sekarang, halaman web sering kali menggunakan **JavaScript dan AJAX** untuk mengambil data dari server dalam format **JSON**, lalu menampilkannya secara dinamis tanpa perlu memuat ulang halaman.
Framework **Pyramid** menyediakan **JSON renderer** yang memudahkan proses ini.

JSON renderer di Pyramid memungkinkan kita mengembalikan data Python dalam format JSON secara otomatis. Selain itu, Pyramid juga mengatur content type (`application/json`) dan memberikan fleksibilitas untuk mengelola API berbasis data.

## Tujuan

* Memahami cara menggunakan JSON renderer di Pyramid
* Membuat endpoint API yang menampilkan data dalam format JSON
* Menerapkan beberapa decorator (`@view_config`) pada satu method view

---

## Langkah-langkah

### 1. Menyalin Proyek Sebelumnya

Gunakan proyek `view_classes` sebagai dasar, lalu salin ke folder baru bernama `json`:

```bash
cd ..; cp -r view_classes json; cd json
$VENV/bin/pip install -e .
```

Langkah ini membuat salinan proyek yang akan kita ubah agar dapat menampilkan data JSON.

---

### 2. Menambahkan Route untuk JSON

Tambahkan route baru `hello_json` di file `json/tutorial/__init__.py`:

```python
from pyramid.config import Configurator


def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.include('pyramid_chameleon')
    config.add_route('home', '/')
    config.add_route('hello', '/howdy')
    config.add_route('hello_json', '/howdy.json')  # Route untuk JSON
    config.scan('.views')
    return config.make_wsgi_app()
```

Baris `config.add_route('hello_json', '/howdy.json')` digunakan untuk membuat endpoint yang akan mengembalikan data JSON dari server.

---

### 3. Menambahkan Decorator JSON di View

Edit file `json/tutorial/views.py`, lalu tambahkan decorator kedua pada method `hello` agar dapat menangani permintaan JSON:

```python
from pyramid.view import (
    view_config,
    view_defaults
    )


@view_defaults(renderer='home.pt')
class TutorialViews:
    def __init__(self, request):
        self.request = request

    @view_config(route_name='home')
    def home(self):
        return {'name': 'Home View'}

    @view_config(route_name='hello')
    @view_config(route_name='hello_json', renderer='json')  # Tambahan untuk JSON
    def hello(self):
        return {'name': 'Hello View'}
```

Penjelasan:

* `@view_config(route_name='hello')` → menghasilkan halaman HTML.
* `@view_config(route_name='hello_json', renderer='json')` → menghasilkan response dalam format JSON.

Satu method `hello()` sekarang bisa melayani dua permintaan berbeda: **HTML** dan **JSON**.

---

### 4. Menambahkan Functional Test

Tambahkan test di akhir file `json/tutorial/tests.py` untuk memastikan endpoint JSON berfungsi dengan benar:

```python
def test_hello_json(self):
    res = self.testapp.get('/howdy.json', status=200)
    self.assertIn(b'{"name": "Hello View"}', res.body)
    self.assertEqual(res.content_type, 'application/json')
```

Test ini memastikan response dari `/howdy.json` benar-benar mengembalikan data JSON dan memiliki content type yang sesuai.

---

### 5. Menjalankan Test

Jalankan seluruh test menggunakan perintah berikut:

```bash
$VENV/bin/pytest tutorial/tests.py -q
```

Jika konfigurasi benar, hasilnya akan seperti ini:

```
.....
5 passed in 0.47 seconds
```

Hasil:
<img width="1239" height="447" alt="Screenshot 2025-11-14 001511" src="https://github.com/user-attachments/assets/b2531cbd-75a2-4dc6-9f9c-a53f2ab8a29e" />

---

### 6. Menjalankan Aplikasi

Jalankan server Pyramid dalam mode pengembangan:

```bash
$VENV/bin/pserve development.ini --reload
```

---

### 7. Melihat Hasil di Browser

Buka alamat berikut di browser:

```
http://localhost:6543/howdy.json
```

Tampilan di browser akan menunjukkan hasil response JSON seperti:

```json
{"name": "Hello View"}
```

Hasil:
<img width="1919" height="1013" alt="Screenshot 2025-11-14 001541" src="https://github.com/user-attachments/assets/f4b0082c-0ac3-4a62-8535-4749ec60411f" />

<img width="1919" height="1014" alt="Screenshot 2025-11-14 001549" src="https://github.com/user-attachments/assets/ef2601c0-a96b-4e15-9f34-f1e617e97652" />

---

## Analisis

### Konsep Data-Oriented View Layer

Sebelumnya, kita telah memisahkan logika view dari tampilan HTML dengan cara mengembalikan data Python. Pendekatan ini membuat kode lebih mudah diuji dan fleksibel.

Dengan menggunakan JSON renderer, Pyramid dapat otomatis mengubah data Python menjadi format JSON tanpa tambahan kode manual.

---

### Cara Kerja JSON Renderer

Langkah-langkah yang dilakukan dalam implementasi ini:

1. Menambahkan route baru `/howdy.json` yang diarahkan ke `hello_json`.
2. Menambahkan decorator kedua pada method `hello()` untuk menangani format JSON.
3. Renderer `json` secara otomatis mengubah data dictionary Python menjadi JSON dan menetapkan `Content-Type: application/json`.

---

### Multiple Decorator (Stacking)

Satu view bisa memiliki beberapa decorator agar dapat menghasilkan output berbeda tergantung route-nya:

```python
@view_config(route_name='hello')  # Mengembalikan HTML
@view_config(route_name='hello_json', renderer='json')  # Mengembalikan JSON
def hello(self):
    return {'name': 'Hello View'}
```

Dengan cara ini:

* Mengakses `/howdy` → tampil sebagai halaman HTML.
* Mengakses `/howdy.json` → tampil sebagai data JSON.

---

### Alternatif: View Predicates

Sebagai alternatif, Pyramid juga mendukung **view predicates** untuk menentukan renderer berdasarkan header `Accept:` dari request.
Dengan begitu, kita bisa menggunakan satu route yang sama tanpa perlu menambahkan `.json` di URL.

---

### Keterbatasan JSON Renderer

JSON renderer di Pyramid menggunakan encoder bawaan dari Python. Akibatnya, beberapa tipe data seperti **datetime** tidak dapat dikonversi secara langsung ke JSON.

**Solusi yang dapat dilakukan:**

* Membuat custom JSON renderer
* Mengubah data datetime ke string sebelum dikirim
* Menggunakan serializer eksternal seperti `simplejson`

Dengan cara ini, developer bisa menyesuaikan renderer sesuai kebutuhan aplikasi.

---

## Kesimpulan

Dalam tutorial ini, kita telah belajar bahwa Pyramid dapat dengan mudah digunakan untuk membuat **API berbasis JSON**.
Langkah-langkah utamanya meliputi:

1. Menambahkan route untuk endpoint JSON
2. Menggunakan decorator tambahan dengan `renderer='json'`
3. Menguji endpoint dengan functional test
4. Menjalankan aplikasi untuk melihat hasilnya di browser

Dengan pendekatan ini, satu view bisa melayani dua format berbeda — **HTML untuk tampilan browser**, dan **JSON untuk komunikasi AJAX** — sehingga membuat aplikasi web lebih interaktif dan modern.
