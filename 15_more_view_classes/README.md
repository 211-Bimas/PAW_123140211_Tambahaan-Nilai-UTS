# Tutorial Pyramid: More With View Classes

## Deskripsi

Tutorial ini membahas cara **mengelompokkan beberapa view ke dalam satu class (view class)** agar dapat berbagi konfigurasi, state (data), dan logika yang sama.
Pendekatan ini membuat kode aplikasi web menjadi **lebih rapi, terstruktur, dan efisien**.

Dalam framework Pyramid, *view* bisa berupa:

* Fungsi biasa
* Objek dengan method `__call__`
* Sebuah class Python, di mana setiap method-nya bisa dijadikan view dengan menggunakan decorator `@view_config`.

Awalnya views dibuat sebagai fungsi yang berdiri sendiri. Namun jika beberapa views menangani data yang sama (seperti pada form input, edit, dan delete), lebih baik digabungkan dalam satu class.

### Keuntungan menggunakan View Class

* Mengelompokkan views yang saling berhubungan
* Memusatkan pengaturan konfigurasi agar tidak berulang
* Mempermudah berbagi data dan fungsi pembantu
* Membuat struktur kode lebih bersih dan mudah dipelihara

Pyramid juga menyediakan **view predicates** yang dapat menentukan view mana yang dipanggil berdasarkan kondisi tertentu, seperti:

* Jenis permintaan HTTP (GET, POST, dsb.)
* Parameter form yang dikirim
* Informasi tambahan dari request

---

## Tujuan

* Menggabungkan beberapa view terkait ke dalam satu class
* Mengatur konfigurasi bersama menggunakan `@view_defaults`
* Mengarahkan satu route ke beberapa view berdasarkan request
* Membagikan state dan logika antara views serta templates

---

## Langkah-Langkah

### 1. Menyalin Proyek Sebelumnya

Gunakan hasil proyek dari langkah templating:

```bash
cd ..; cp -r templating more_view_classes; cd more_view_classes
$VENV/bin/pip install -e .
```

### 2. Menambahkan Route

Edit file `more_view_classes/tutorial/__init__.py`:

```python
from pyramid.config import Configurator

def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.include('pyramid_chameleon')
    config.add_route('home', '/')
    config.add_route('hello', '/howdy/{first}/{last}')
    config.scan('.views')
    return config.make_wsgi_app()
```

### 3. Membuat View Class

File: `more_view_classes/tutorial/views.py`

```python
from pyramid.view import view_config, view_defaults

@view_defaults(route_name='hello')
class TutorialViews:
    def __init__(self, request):
        self.request = request
        self.view_name = 'TutorialViews'

    @property
    def full_name(self):
        first = self.request.matchdict['first']
        last = self.request.matchdict['last']
        return first + ' ' + last

    @view_config(route_name='home', renderer='home.pt')
    def home(self):
        return {'page_title': 'Home View'}

    @view_config(renderer='hello.pt')
    def hello(self):
        return {'page_title': 'Hello View'}

    @view_config(request_method='POST', renderer='edit.pt')
    def edit(self):
        new_name = self.request.params['new_name']
        return {'page_title': 'Edit View', 'new_name': new_name}

    @view_config(request_method='POST', request_param='form.delete', renderer='delete.pt')
    def delete(self):
        print('Deleted')
        return {'page_title': 'Delete View'}
```

### 4. Template Home

File: `more_view_classes/tutorial/home.pt`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>${view.view_name} - ${page_title}</title>
</head>
<body>
<h1>${view.view_name} - ${page_title}</h1>
<p>Go to the <a href="${request.route_url('hello', first='jane', last='doe')}">form</a>.</p>
</body>
</html>
```

### 5. Template Hello

File: `more_view_classes/tutorial/hello.pt`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>${view.view_name} - ${page_title}</title>
</head>
<body>
<h1>${view.view_name} - ${page_title}</h1>
<p>Welcome, ${view.full_name}</p>
<form method="POST" action="${request.current_route_url()}">
    <input name="new_name"/>
    <input type="submit" name="form.edit" value="Save"/>
    <input type="submit" name="form.delete" value="Delete"/>
</form>
</body>
</html>
```

### 6. Template Edit

File: `more_view_classes/tutorial/edit.pt`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>${view.view_name} - ${page_title}</title>
</head>
<body>
<h1>${view.view_name} - ${page_title}</h1>
<p>You submitted <code>${new_name}</code></p>
</body>
</html>
```

### 7. Template Delete

File: `more_view_classes/tutorial/delete.pt`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>${view.view_name} - ${page_title}</title>
</head>
<body>
<h1>${view.view_name} - ${page_title}</h1>
</body>
</html>
```

### 8. Mengedit File Test

File: `more_view_classes/tutorial/tests.py`

```python
import unittest
from pyramid import testing

class TutorialViewTests(unittest.TestCase):
    def setUp(self):
        self.config = testing.setUp()
    def tearDown(self):
        testing.tearDown()
    def test_home(self):
        from .views import TutorialViews
        request = testing.DummyRequest()
        inst = TutorialViews(request)
        response = inst.home()
        self.assertEqual('Home View', response['page_title'])

class TutorialFunctionalTests(unittest.TestCase):
    def setUp(self):
        from tutorial import main
        app = main({})
        from webtest import TestApp
        self.testapp = TestApp(app)
    def test_home(self):
        res = self.testapp.get('/', status=200)
        self.assertIn(b'TutorialViews - Home View', res.body)
```

### 9. Menjalankan Test

Gunakan perintah berikut:

```bash
$VENV/bin/pytest tutorial/tests.py -q
```

Hasil yang diharapkan:

```
2 passed in 0.40 seconds
```

Hasil:
<img width="1236" height="443" alt="Screenshot 2025-11-14 002808" src="https://github.com/user-attachments/assets/0d9276c1-e283-4f7c-b323-e63ab8f8ae95" />

### 10. Menjalankan Aplikasi

```bash
$VENV/bin/pserve development.ini --reload
```

### 11. Melihat Hasil di Browser

Buka [http://localhost:6543/howdy/jane/doe](http://localhost:6543/howdy/jane/doe)
Klik **Save** atau **Delete**, lalu lihat hasilnya di terminal.

Hasil:
<img width="1919" height="1021" alt="Screenshot 2025-11-14 002840" src="https://github.com/user-attachments/assets/dc8e3853-5175-4ab5-9acf-47d4bde5eb09" />

<img width="1919" height="1020" alt="Screenshot 2025-11-14 002854" src="https://github.com/user-attachments/assets/b1bd7a58-a2a5-479e-b8e8-65cb4f326718" />

---

## Analisis

### 1. Pengelompokan View

Empat views saling terhubung:

1. **Home View** – diakses dari `/`
2. **Hello View** – diakses dari `/howdy/jane/doe`
3. **Edit View** – dipanggil saat form dikirim (POST)
4. **Delete View** – dipanggil saat tombol Delete ditekan

### 2. View Predicates

View dipilih berdasarkan:

* Jenis permintaan HTTP (`GET`, `POST`)
* Parameter yang dikirim (`form.delete`, dsb.)

### 3. Konfigurasi Terpusat

Dengan `@view_defaults`, semua view dalam class bisa memiliki pengaturan dasar yang sama dan hanya diubah jika diperlukan.

### 4. Berbagi State dan Logic

Nilai seperti `view_name` dan `full_name` bisa digunakan oleh semua method dan juga diakses di template, sehingga kode lebih efisien.

### 5. Dynamic URL Generation

Daripada menulis URL manual:

```html
<a href="/howdy/jane/doe">Howdy</a>
```

Gunakan cara dinamis:

```html
<a href="${request.route_url('hello', first='jane', last='doe')}">form</a>
```

Keuntungan:

* Lebih fleksibel jika route berubah
* Lebih aman dan mudah dirawat

---

## Ekstra Kredit

### 1. Mengapa Bisa Mengakses `${view.full_name}` Tanpa `()`?

Karena `full_name` menggunakan decorator `@property`.
Dengan property, method bisa dipanggil seperti atribut biasa tanpa tanda kurung.
Contoh:

```python
@property
def full_name(self):
    return first + ' ' + last
```

### 2. Mengapa Edit View Tidak Menangkap POST dari Delete?

Keduanya menggunakan `POST`, tetapi **Delete View** memiliki syarat tambahan (`request_param='form.delete'`).
Karena itu Pyramid akan memilih Delete View jika parameter tersebut ada.
Pyramid akan menjalankan view dengan konfigurasi yang paling spesifik.

### 3. Apakah Nilai Property Bisa Di-cache?

Ya. Pyramid menyediakan decorator `@reify` untuk menyimpan hasil perhitungan pertama agar tidak dihitung ulang:

```python
from pyramid.decorator import reify

@reify
def full_name(self):
    return first + ' ' + last
```

Perbedaan:

* `@property`: dihitung ulang setiap kali diakses
* `@reify`: dihitung sekali, lalu disimpan

### 4. Dapatkah Satu View Memiliki Beberapa Route?

Bisa. Satu view dapat dihubungkan ke beberapa route:

```python
@view_config(route_name='hello', renderer='hello.pt')
@view_config(route_name='greet', renderer='hello.pt')
def hello(self):
    return {'page_title': 'Hello View'}
```

### 5. Perbedaan `route_path` dan `route_url`

* **`request.route_url`** menghasilkan URL lengkap (dengan domain), contoh:

  ```
  http://localhost:6543/howdy/jane/doe
  ```
* **`request.route_path`** hanya menghasilkan path:

  ```
  /howdy/jane/doe
  ```

Gunakan `route_url` untuk tautan keluar (misalnya email), dan `route_path` untuk navigasi internal di aplikasi.

---

## Kesimpulan

Dengan menggunakan **View Classes**, aplikasi Pyramid menjadi:

1. Lebih terstruktur dan mudah dibaca
2. Konfigurasinya terpusat dengan `@view_defaults`
3. Logika bisa dibagi antar views
4. Routing lebih fleksibel dengan view predicates
5. URL lebih aman dan dinamis

Pendekatan ini sangat berguna untuk proyek besar agar pengelolaan kode tetap rapi dan konsisten.
