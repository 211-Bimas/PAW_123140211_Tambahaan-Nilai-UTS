# **Membuat HTML dengan Templating di Pyramid**

## **Deskripsi**

Pada tutorial ini, kita akan mempelajari cara menggunakan **sistem templating** untuk menghasilkan HTML di framework **Pyramid**.
Dengan sistem ini, kita tidak perlu lagi menulis kode HTML langsung di dalam Python. Sebagai gantinya, kita akan memisahkan **logika program** dan **tampilan** ke dalam file yang berbeda agar struktur aplikasi lebih rapi dan mudah dikelola.

Sebelumnya, kita membuat HTML secara langsung di dalam fungsi *view*. Pendekatan ini tidak efisien dan sulit dipelihara saat proyek bertambah besar.
Framework web modern biasanya menggunakan sistem templating agar lebih terorganisir.

Pyramid mendukung berbagai *template engine* seperti **Jinja2**, **Mako**, dan **Chameleon**. Dalam tutorial ini, kita akan menggunakan **pyramid_chameleon** sebagai contoh.

---

## **Tujuan Pembelajaran**

1. Mengaktifkan *add-on* `pyramid_chameleon` pada proyek Pyramid.
2. Menghasilkan HTML menggunakan file template terpisah.
3. Menghubungkan template sebagai *renderer* untuk *view*.
4. Mengubah *view* agar hanya mengembalikan data, bukan HTML langsung.

---

## **Langkah-langkah Instalasi**

### **1. Membuat Proyek Baru**

Gunakan proyek sebelumnya sebagai dasar, lalu buat salinan baru untuk tahap templating:

```bash
cd ..; cp -r views templating; cd templating
```

---

### **2. Menambahkan Dependency**

Tambahkan *dependency* `pyramid_chameleon` ke dalam file `templating/setup.py` agar Pyramid bisa menggunakan sistem templating.

```python
from setuptools import setup

requires = [
    'pyramid',
    'pyramid_chameleon',
    'waitress',
]

dev_requires = [
    'pyramid_debugtoolbar',
    'pytest',
    'webtest',
]

setup(
    name='tutorial',
    install_requires=requires,
    extras_require={
        'dev': dev_requires,
    },
    entry_points={
        'paste.app_factory': [
            'main = tutorial:main'
        ],
    },
)
```

---

### **3. Menginstal Dependency**

Setelah menambahkan konfigurasi di atas, aktifkan *environment* pengembangan dan jalankan perintah berikut:

```bash
$VENV/bin/pip install -e .
```

---

### **4. Konfigurasi Template Engine**

Aktifkan **pyramid_chameleon** dengan menambahkan baris berikut di `tutorial/__init__.py`:

```python
from pyramid.config import Configurator

def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.include('pyramid_chameleon')
    config.add_route('home', '/')
    config.add_route('hello', '/howdy')
    config.scan('.views')
    return config.make_wsgi_app()
```

Fungsi `config.include('pyramid_chameleon')` akan menghubungkan Pyramid dengan sistem templating Chameleon.

---

### **5. Membuat View**

Selanjutnya, ubah `tutorial/views.py` agar tidak lagi menulis HTML secara langsung.
Setiap *view* cukup mengembalikan data dalam bentuk dictionary, sementara tampilan HTML akan dihasilkan oleh template.

```python
from pyramid.view import view_config

@view_config(route_name='home', renderer='home.pt')
def home(request):
    return {'name': 'Home View'}

@view_config(route_name='hello', renderer='home.pt')
def hello(request):
    return {'name': 'Hello View'}
```

Kedua *view* di atas menggunakan template yang sama, yaitu `home.pt`.

---

### **6. Membuat Template**

Buat file baru bernama `tutorial/home.pt` dengan isi berikut:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Tutorial Pyramid: ${name}</title>
</head>
<body>
    <h1>Hi ${name}</h1>
</body>
</html>
```

**Penjelasan:**

* `${name}` akan otomatis diganti dengan data yang dikirim dari fungsi *view*.
* File ini menggunakan sintaks dari **Chameleon Template Engine**.

---

### **7. Konfigurasi Development Mode**

Agar template otomatis *reload* saat diedit, tambahkan pengaturan berikut di file `development.ini`:

```
[app:main]
use = egg:tutorial
pyramid.reload_templates = true
pyramid.includes =
    pyramid_debugtoolbar

[server:main]
use = egg:waitress#main
listen = localhost:6543
```

Dengan pengaturan ini, kamu tidak perlu me-*restart* server setiap kali mengubah template.

---

## **Testing**

### **Unit Test**

Sekarang, pengujian hanya perlu memastikan bahwa *view* mengembalikan data yang benar, bukan HTML.

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
        self.assertEqual('Home View', response['name'])

    def test_hello(self):
        from .views import hello
        request = testing.DummyRequest()
        response = hello(request)
        self.assertEqual('Hello View', response['name'])
```

---

### **Functional Test**

Tambahkan juga pengujian untuk memastikan HTML dihasilkan dengan benar oleh template:

```python
class TutorialFunctionalTests(unittest.TestCase):
    def setUp(self):
        from tutorial import main
        app = main({})
        from webtest import TestApp
        self.testapp = TestApp(app)

    def test_home(self):
        res = self.testapp.get('/', status=200)
        self.assertIn(b'<h1>Hi Home View', res.body)

    def test_hello(self):
        res = self.testapp.get('/howdy', status=200)
        self.assertIn(b'<h1>Hi Hello View', res.body)
```

---

### **Menjalankan Test**

Gunakan perintah berikut:

```bash
$VENV/bin/pytest tutorial/tests.py -q
```

Hasil yang diharapkan:

```
....
4 passed in 0.46 seconds
```

Hasil :
<img width="1241" height="446" alt="Screenshot 2025-11-13 224536" src="https://github.com/user-attachments/assets/cd299396-e9e7-4cfb-88fc-99dbc917b1c3" />

---

### **Menjalankan Aplikasi**

Jalankan server Pyramid:

```bash
$VENV/bin/pserve development.ini --reload
```

Buka di browser:

* [http://localhost:6543/](http://localhost:6543/)
* [http://localhost:6543/howdy](http://localhost:6543/howdy)

Hasil: 
<img width="1919" height="1019" alt="Screenshot 2025-11-13 224602" src="https://github.com/user-attachments/assets/e662dd66-c93f-49fe-a93d-222d0fe8c599" />

<img width="1919" height="1019" alt="Screenshot 2025-11-13 224627" src="https://github.com/user-attachments/assets/5108ef9e-b718-48c6-b35e-c46b19acb337" />

---

## **Analisis**

Setelah menggunakan sistem templating, kode menjadi jauh lebih bersih.
Fungsi *view* kini hanya fokus pada **logika dan data**, sedangkan tampilan HTML dikelola oleh template.

Perhatikan bahwa kita menggunakan **satu template yang sama** untuk dua *view* berbeda (`home` dan `hello`).
Pendekatan ini membuat kode lebih efisien dan mudah dikelola.

---

### **Keuntungan Menggunakan Templating**

1. **Pemisahan Tanggung Jawab (Separation of Concerns)**

   * *View* berisi logika program
   * Template mengatur tampilan HTML
   * Kode lebih terstruktur dan mudah dikembangkan

2. **Dapat Digunakan Ulang (Reusable)**

   * Satu file template bisa digunakan untuk banyak *view*
   * Menghemat waktu dan mencegah duplikasi kode

3. **Testing Lebih Mudah**

   * Pengujian fokus pada data, bukan HTML mentah
   * Test menjadi lebih sederhana dan tahan terhadap perubahan tampilan

4. **Fleksibilitas Tinggi**

   * Pyramid mendukung berbagai *template engine*
   * Developer bebas menggunakan Chameleon, Jinja2, atau Mako

5. **Pengalaman Pengembangan Lebih Baik**

   * Template otomatis *reload* saat diedit
   * Tidak perlu me-*restart* server setiap kali mengubah tampilan

---

### **Konsep Penting**

**Renderer:**

* Menghubungkan fungsi *view* dengan template HTML
* Data dari *view* (dalam bentuk dictionary) otomatis diteruskan ke template

**Data Contract:**

* *View* hanya mengembalikan data
* Template menampilkan data tersebut di halaman web
* Menciptakan kontrak yang jelas antara logika dan tampilan

---

## **Kesimpulan**

Menggunakan sistem templating membuat struktur aplikasi **lebih bersih, terpisah, dan mudah dipelihara**.
*View* hanya fokus pada pengolahan data, sedangkan *template* bertanggung jawab terhadap tampilan.

Pendekatan ini adalah praktik umum dalam pengembangan web modern karena meningkatkan efisiensi, keterbacaan, dan kemudahan pengujian aplikasi.

