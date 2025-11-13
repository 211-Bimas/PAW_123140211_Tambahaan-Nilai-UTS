# Penanganan Web Dasar dengan Views di Pyramid**

Mengatur modul *views* menggunakan *decorator* dan beberapa *view* sekaligus.

---

## **Deskripsi**

Dalam framework Pyramid, **view** adalah bagian utama yang menerima *web request* dan mengembalikan *response*.
Pada contoh sebelumnya, fungsi `hello_world` berperan sebagai *view* sederhana.

Sampai tahap ini, seluruh kode aplikasi — mulai dari fungsi *view*, konfigurasi, *routing*, hingga *launcher* WSGI — masih berada dalam satu file.
Struktur seperti itu bisa berantakan jika proyek semakin besar.

Untuk memperbaikinya, kita akan memindahkan *views* ke file terpisah bernama `views.py`.
Kita juga akan menambahkan *decorator* agar konfigurasi menjadi **deklaratif**, lalu menambah satu *view* baru serta memperbarui pengujiannya.

---

## **Tujuan**

* Memindahkan *view* ke modul terpisah agar lebih terorganisir.
* Menggunakan *decorator* sebagai bentuk konfigurasi deklaratif.
* Menambahkan *view* tambahan dan melakukan pengujian pada keduanya.

---

## **Langkah-Langkah**

### **1. Menyalin Proyek Sebelumnya**

Gunakan proyek dari tahap sebelumnya sebagai dasar, lalu buat salinannya:

```bash
cd ..; cp -r functional_testing views; cd views
$VENV/bin/pip install -e .
```

---

### **2. Menyederhanakan `__init__.py`**

Isi file `tutorial/__init__.py` menjadi lebih singkat seperti berikut:

```python
from pyramid.config import Configurator

def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.add_route('home', '/')
    config.add_route('hello', '/howdy')
    config.scan('.views')
    return config.make_wsgi_app()
```

**Penjelasan singkat:**

* Dua *route* dibuat, masing-masing untuk halaman utama (`/`) dan halaman `/howdy`.
* `config.scan('.views')` digunakan agar Pyramid mencari *decorator* `@view_config` di file `views.py`.

---

### **3. Membuat Modul `views.py`**

Tambahkan file baru `tutorial/views.py` untuk menampung fungsi-fungsi *view*.

```python
from pyramid.response import Response
from pyramid.view import view_config

# View pertama (halaman utama)
@view_config(route_name='home')
def home(request):
    return Response('<body>Visit <a href="/howdy">hello</a></body>')

# View kedua (/howdy)
@view_config(route_name='hello')
def hello(request):
    return Response('<body>Go back <a href="/">home</a></body>')
```

---

### **4. Memperbarui File Test**

Tambahkan atau ubah file `tutorial/tests.py` agar mencakup dua *view* di atas.

```python
import unittest
from pyramid import testing

class TutorialViewTests(unittest.TestCase):
    def setUp(self):
        self.config = testing.setUp()

    def tearDown(self):
        testing.tearDown()

    def test_home(self):
        from .views import home
        request = testing.DummyRequest()
        response = home(request)
        self.assertEqual(response.status_code, 200)
        self.assertIn(b'Visit', response.body)

    def test_hello(self):
        from .views import hello
        request = testing.DummyRequest()
        response = hello(request)
        self.assertEqual(response.status_code, 200)
        self.assertIn(b'Go back', response.body)

class TutorialFunctionalTests(unittest.TestCase):
    def setUp(self):
        from tutorial import main
        app = main({})
        from webtest import TestApp
        self.testapp = TestApp(app)

    def test_home(self):
        res = self.testapp.get('/', status=200)
        self.assertIn(b'<body>Visit', res.body)

    def test_hello(self):
        res = self.testapp.get('/howdy', status=200)
        self.assertIn(b'<body>Go back', res.body)
```

---

### **5. Menjalankan Test**

Gunakan perintah berikut:

```bash
$VENV/bin/pytest tutorial/tests.py -q
```

Jika konfigurasi benar, hasilnya akan muncul seperti ini:

```
....
4 passed in 0.28 seconds
```

Outputnya :
<img width="1240" height="446" alt="Screenshot 2025-11-13 205521" src="https://github.com/user-attachments/assets/05129b4e-ff45-41bb-ac67-01988f776f31" />

---

### **6. Menjalankan Aplikasi**

Jalankan server Pyramid:

```bash
$VENV/bin/pserve development.ini --reload
```

Buka di browser:

* [http://localhost:6543/](http://localhost:6543/)
* [http://localhost:6543/howdy](http://localhost:6543/howdy)

---

Outputnya :
<img width="1919" height="1017" alt="image" src="https://github.com/user-attachments/assets/64278c0f-edd0-494a-8833-bf5abbff758f" />

<img width="1919" height="1020" alt="Screenshot 2025-11-13 205700" src="https://github.com/user-attachments/assets/f03ab297-def1-4d19-bb76-4b7e937265d0" />

## **Analisis**

Dalam tahap ini, kita menambahkan beberapa URL baru sekaligus memisahkan kode *view* dari konfigurasi aplikasi di `__init__.py`.
Sekarang, semua *view* dan dekoratornya berada di `views.py`, dan secara otomatis terdaftar melalui `config.scan('.views')`.

Setiap *view* saling terhubung:

* `home` menampilkan tautan menuju `/howdy`
* `hello` memiliki tautan kembali ke `/`

Hal ini menunjukkan bahwa nama *route*, URL, dan nama fungsi *view* **tidak harus sama**, tetapi bisa dihubungkan melalui konfigurasi.

Sebelumnya, kita menggunakan `config.add_view` (konfigurasi **imperatif**) untuk mendaftarkan *view*.
Sekarang kita beralih ke `@view_config` (konfigurasi **deklaratif**), di mana dekorator langsung menunjukkan fungsi mana yang menjadi *view*.
Keduanya menghasilkan hasil yang sama — perbedaannya hanya pada gaya penulisan dan kemudahan pemeliharaan.

---

## **Extra Credit**

### **1. Apa Arti Titik (.) pada `.views`?**

Tanda titik (`.`) pada `config.scan('.views')` menunjukkan **import relatif**, yaitu:

* `.` berarti *package saat ini* (`tutorial`)
* `views` merujuk pada file `views.py` di dalam package tersebut
  Jadi, `config.scan('.views')` berarti *scan module tutorial.views*.

Keuntungan pendekatan ini:

* Lebih **portabel** (tidak perlu mengganti nama package jika berubah)
* Lebih **jelas** dan **ringkas**
* Sesuai dengan *konvensi resmi* Pyramid

---

### **2. Mengapa `assertIn` Lebih Disarankan daripada `assertEqual` dalam Test?**

`assertIn` digunakan untuk memeriksa apakah potongan teks tertentu ada di dalam *response body*.
Sementara `assertEqual` membandingkan teks secara keseluruhan.

**Perbandingan:**

```python
# Lebih fleksibel
self.assertIn(b'Visit', response.body)

# Kurang fleksibel (harus cocok 100%)
self.assertEqual(response.body, b'<body>Visit <a href="/howdy">hello</a></body>')
```

`assertIn` lebih disarankan karena:

* Tetap valid meskipun ada sedikit perubahan HTML.
* Lebih mudah dirawat saat desain tampilan berubah.
* Fokus pada konten penting, bukan detail kecil seperti spasi atau atribut tambahan.

---

## **Kesimpulan**

Dengan memindahkan *view* ke modul terpisah dan menggunakan dekorator `@view_config`, kode menjadi lebih:

* **Terstruktur** — setiap bagian punya tanggung jawab yang jelas.
* **Deklaratif** — konfigurasi dilakukan langsung di fungsi *view*.
* **Mudah dikelola** — Pyramid otomatis menemukan semua *view* melalui `config.scan()`.

Pendekatan ini tidak hanya membuat kode lebih bersih, tetapi juga membantu memahami bagaimana *request* diterima, diproses, dan dikembalikan menjadi *response* di dalam aplikasi web Pyramid.

