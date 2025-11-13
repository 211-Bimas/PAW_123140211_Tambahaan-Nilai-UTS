# Mengorganisir Views dengan View Classes

## Deskripsi

Tutorial ini menjelaskan cara mengubah *view functions* menjadi *view classes* pada **Pyramid Framework**.
Dengan menggunakan *view classes*, beberapa *view* yang saling berhubungan dapat digabungkan ke dalam satu kelas, sehingga kode menjadi lebih rapi, mudah dikelola, dan efisien.

Sebelumnya, semua *view* dibuat dalam bentuk fungsi yang terpisah.
Namun dalam aplikasi nyata, sering kali beberapa *view* bekerja dengan data yang sama atau memiliki tujuan yang mirip, misalnya:

* Menampilkan data dari sumber yang sama
* Menangani berbagai operasi pada REST API
* Membutuhkan konfigurasi atau fungsi bantu (*helper function*) yang sama

### Keuntungan Menggunakan View Classes

1. **Mengelompokkan Views yang Berkaitan**
   Semua *view* dengan fungsi serupa dapat disatukan dalam satu kelas.
2. **Menyederhanakan Konfigurasi**
   Pengaturan yang berulang dapat dipusatkan di tingkat kelas.
3. **Berbagi Data dan Fungsi Bantu**
   Data atau fungsi yang sama bisa digunakan oleh beberapa *view* di dalam satu kelas.

---

## Tujuan

1. Mengelompokkan *view* yang berhubungan ke dalam satu *class*.
2. Menyatukan pengaturan menggunakan `@view_defaults`.
3. Memahami bagaimana *view class* dibuat dan digunakan di Pyramid.

---

## Langkah-langkah Implementasi

### 1. Menyalin Proyek Sebelumnya

```bash
cd ..; cp -r templating view_classes; cd view_classes
$VENV/bin/pip install -e .
```

---

### 2. Membuat View Class

Pada file `view_classes/tutorial/views.py`, ubah fungsi *view* menjadi metode di dalam sebuah kelas.

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
    def hello(self):
        return {'name': 'Hello View'}
```

#### Penjelasan Kode

* **`@view_defaults(renderer='home.pt')`**
  Digunakan untuk memberikan pengaturan umum di tingkat kelas.
  Semua metode di dalam kelas ini otomatis memakai renderer `home.pt`, sehingga tidak perlu ditulis berulang di setiap `@view_config`.

* **`__init__(self, request)`**
  Fungsi *constructor* yang dijalankan otomatis oleh Pyramid.
  Parameter `request` berisi informasi permintaan (seperti data user, URL, dan lain-lain).

* **Metode `home()` dan `hello()`**
  Adalah dua *view* yang sebelumnya berbentuk fungsi, kini menjadi metode dalam satu kelas.
  Keduanya mengembalikan *dictionary* yang akan digunakan oleh template.

---

### 3. Memperbarui Unit Test

Edit file `view_classes/tutorial/tests.py` agar menggunakan *view class*.

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
        self.assertEqual('Home View', response['name'])

    def test_hello(self):
        from .views import TutorialViews
        request = testing.DummyRequest()
        inst = TutorialViews(request)
        response = inst.hello()
        self.assertEqual('Hello View', response['name'])
```

#### Pola Pengujian View Class

1. Import kelas *view*.
2. Buat *DummyRequest* untuk meniru permintaan pengguna.
3. Buat instance dari kelas tersebut.
4. Jalankan metode yang ingin diuji dan periksa hasilnya.

---

### 4. Menjalankan Test

```bash
$VENV/bin/pytest tutorial/tests.py -q
```

Jika berhasil, hasilnya akan seperti ini:

```
....
4 passed in 0.34 seconds
```

Hasil :
<img width="1244" height="447" alt="Screenshot 2025-11-13 230352" src="https://github.com/user-attachments/assets/f562d35e-5108-47b5-84d2-5ecbf7a2d63e" />

---

### 5. Menjalankan Aplikasi

```bash
$VENV/bin/pserve development.ini --reload
```

Buka browser dan akses:

* [http://localhost:6543/](http://localhost:6543/)
* [http://localhost:6543/howdy](http://localhost:6543/howdy)

**Output yang ditampilkan di browser:**

```
Hi Home View
Hi Hello View
```

Hasil:
<img width="1919" height="1019" alt="Screenshot 2025-11-13 230425" src="https://github.com/user-attachments/assets/ba9f68c9-df16-4c81-aaae-f966553cf899" />

<img width="1919" height="1019" alt="Screenshot 2025-11-13 230439" src="https://github.com/user-attachments/assets/b416aeeb-550b-4483-a65b-a18fbc568723" />

---

## Analisis

Pada tahap ini, tidak ada fitur baru yang ditambahkan.
Kita hanya mengubah struktur kode agar lebih terorganisir.

Sebelumnya, setiap *view* berdiri sendiri, tetapi sekarang dua *view* tersebut digabungkan ke dalam satu kelas dengan `@view_defaults`.
Hal ini membuat kode lebih mudah dibaca dan dirawat.

### Sebelum (View Functions)

```python
@view_config(route_name='home', renderer='home.pt')
def home(request):
    return {'name': 'Home View'}

@view_config(route_name='hello', renderer='home.pt')
def hello(request):
    return {'name': 'Hello View'}
```

**Masalah:**

* Pengaturan `renderer` ditulis berulang.
* Setiap *view* terpisah, meskipun fungsinya mirip.
* Sulit berbagi data atau fungsi bantu.

### Sesudah (View Class)

```python
@view_defaults(renderer='home.pt')
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

**Keuntungan:**

* Konfigurasi umum hanya ditulis sekali.
* Semua *view* yang berkaitan berada dalam satu tempat.
* Dapat berbagi data atau metode bantu dengan mudah.

---

## Konsep Penting

### 1. View Defaults

`@view_defaults` digunakan untuk memberikan pengaturan umum pada semua metode dalam satu kelas.
Misalnya:

* `renderer` → menentukan template default.
* `permission` → menentukan hak akses default.

### 2. Dependency Injection

Pyramid otomatis memberikan `request` ke dalam *constructor*, sehingga setiap metode bisa mengakses data permintaan melalui `self.request`.

### 3. Instance Variables

Kita dapat menyimpan informasi di dalam kelas agar bisa digunakan oleh beberapa metode:

```python
def __init__(self, request):
    self.request = request
    self.user = request.authenticated_userid
```

---

## Kapan Sebaiknya Menggunakan View Class?

Gunakan *view class* jika:

* Beberapa *view* menggunakan data yang sama.
* Membuat REST API dengan berbagai operasi (GET, POST, PUT, DELETE).
* Ada konfigurasi atau kode yang berulang.
* Ingin menulis kode yang lebih rapi dan mudah diuji.

Gunakan *view function* jika:

* *View* sederhana dan berdiri sendiri.
* Tidak ada konfigurasi berulang.
* Kode kecil dan tidak saling bergantung.

---

## Catatan Penting

1. Gunakan nama kelas dengan akhiran `Views` (contoh: `UserViews`, `AdminViews`).
2. Satu kelas sebaiknya menangani satu fitur utama.
3. Hindari operasi berat di dalam `__init__()`.
4. Saat pengujian, selalu buat instance menggunakan `DummyRequest`.
5. Tambahkan komentar untuk menjelaskan fungsi setiap metode.

---

## Kesimpulan

Menggunakan *view classes* membuat kode Pyramid lebih terstruktur dan mudah dirawat.
Dengan `@view_defaults`, pengaturan berulang bisa dipusatkan, dan berbagai *view* yang saling berkaitan dapat dikelompokkan dalam satu kelas.

Langkah ini mungkin terlihat kecil, tetapi sangat berguna ketika aplikasi berkembang menjadi lebih kompleks.
