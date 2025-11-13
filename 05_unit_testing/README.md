# Unit Testing dengan pytest di Pyramid

## Deskripsi

Tutorial ini menjelaskan cara menulis **unit test** untuk memastikan kode pada aplikasi Pyramid bekerja sebagaimana mestinya menggunakan **pytest**.

Unit testing adalah pengujian terhadap bagian terkecil dari kode (biasanya fungsi atau method) secara terpisah. Tujuannya adalah memastikan setiap unit berfungsi dengan benar tanpa bergantung pada bagian lain.

### Apa itu Unit Test?

* **Unit test** = Pengujian mandiri untuk setiap komponen kecil (fungsi, class, dll)
* **Tujuan utama** = Deteksi dini terhadap bug dan menjaga agar perubahan kode tidak merusak fungsi yang sudah ada
* **pytest** = Framework Python yang lebih ringkas dan kuat dibandingkan `unittest` bawaan

### Struktur Pengembangan

Dalam tutorial ini, pengujian dilakukan di dalam struktur proyek Pyramid dengan penambahan `pytest` ke dalam `setup.py`:

1. **setup.py** – ditambahkan `extras_require` untuk dependency `pytest`
2. **tests.py** – berisi kode unit test
3. **Development mode** – project diinstall dengan opsi `[dev]` agar dependency testing ikut terpasang

* Unit test dijalankan menggunakan `pytest`
* Setiap test dijalankan secara independen (isolasi penuh)

---

## Objektif

* Menulis **unit test** sederhana untuk memastikan view berfungsi dengan benar
* Menambahkan **pytest** ke dalam dependency proyek
* Menjalankan test menggunakan perintah `pytest`

---

## Langkah-langkah

### 1. Tambahkan pytest ke `setup.py`

```python
from setuptools import setup

requires = [
    'pyramid',
    'waitress',
]

dev_requires = [
    'pyramid_debugtoolbar',
    'pytest',
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

### 2. Install Project dalam Development Mode

```bash
$VENV/bin/pip install -e ".[dev]"
```

Perintah ini menginstall package beserta semua dependency tambahan yang didefinisikan di `[dev]`.

### 3. Buat File `tests.py`

```python
import unittest
from pyramid import testing


class TutorialViewTests(unittest.TestCase):
    def setUp(self):
        self.config = testing.setUp()

    def tearDown(self):
        testing.tearDown()

    def test_hello_world(self):
        from tutorial import hello_world

        request = testing.DummyRequest()
        response = hello_world(request)
        self.assertEqual(response.status_code, 200)
```

### 4. Jalankan Test

```bash
$VENV/bin/pytest tutorial/tests.py -q
```

### Output

<img width="1218" height="416" alt="Screenshot 2025-11-13 203402" src="https://github.com/user-attachments/assets/18e5c2b0-0a5c-444a-9bce-56ae3e6753e8" />

---

## ANALISIS

Bagian ini menjelaskan struktur kode test dan alasan di balik pendekatannya.

### Struktur Class Test

```python
class TutorialViewTests(unittest.TestCase):
```

Class ini menggunakan `unittest.TestCase` sebagai dasar, yang memungkinkan kita mendefinisikan method `setUp`, `tearDown`, dan beberapa method test lainnya.

### setUp() dan tearDown()

```python
def setUp(self):
    self.config = testing.setUp()

def tearDown(self):
    testing.tearDown()
```

* **setUp()**: dipanggil sebelum setiap test dijalankan untuk menyiapkan lingkungan uji.
* **tearDown()**: membersihkan lingkungan setelah test selesai.
* Dalam contoh ini, keduanya tidak wajib, tapi bermanfaat untuk test yang lebih kompleks.

### Test Method

```python
def test_hello_world(self):
    from tutorial import hello_world

    request = testing.DummyRequest()
    response = hello_world(request)
    self.assertEqual(response.status_code, 200)
```

**Langkah-langkahnya:**

1. Import fungsi `hello_world` di dalam method (bukan di atas file)
2. Membuat request palsu menggunakan `DummyRequest()`
3. Memanggil fungsi view dengan request tersebut
4. Mengecek apakah hasil respon memiliki status code `200`

**Alasan import dilakukan di dalam method:**

* Mencegah efek samping saat module diimport
* Menjaga setiap test tetap terisolasi
* Memastikan test berjalan independen dari test lain

### Assertion

```python
self.assertEqual(response.status_code, 200)
```

Digunakan untuk memastikan nilai aktual (`response.status_code`) sama dengan nilai yang diharapkan (`200`).

---

## EXTRA CREDIT

### 1. Ubah Status Code yang Diharapkan

Coba ubah assertion menjadi:

```python
self.assertEqual(response.status_code, 404)
```

Kemudian jalankan kembali pytest.
Akan muncul error report yang menunjukkan bahwa nilai yang diharapkan (`404`) berbeda dari hasil aktual (`200`).

### 2. Simulasi Error di View

Tambahkan kesalahan dalam fungsi view seperti berikut:

```python
def hello_world(request):
    x = undefined_variable  # Error!
    return Response('<h1>Hello World!</h1>')
```

Saat dijalankan, pytest akan menampilkan error detail yang memudahkan debugging tanpa membuka browser.

### 3. Ubah Response Code di View

```python
from pyramid.response import Response

def hello_world(request):
    response = Response('<h1>Hello World!</h1>')
    response.status_code = 404
    return response
```

Kemudian jalankan test untuk memastikan kode berubah sesuai “kontrak” logika aplikasi.

### 4. Uji Isi HTML dari Response Body

```python
def test_hello_world_content(self):
    from tutorial import hello_world

    request = testing.DummyRequest()
    response = hello_world(request)

    self.assertIn(b'<h1>Hello World!</h1>', response.body)
    self.assertIn('Hello World!', response.text)
```

---

## Keuntungan Unit Testing

1. **Feedback cepat** – Tidak perlu menjalankan aplikasi manual
2. **Deteksi bug dini** – Kesalahan kecil terdeteksi lebih awal
3. **Dokumentasi kode hidup** – Test menjadi dokumentasi perilaku sistem
4. **Aman saat refactoring** – Bisa mengubah kode tanpa khawatir merusak logika lama
5. **Menjamin konsistensi** – Test memastikan kode berperilaku sesuai harapan

---

## Kesimpulan

Unit testing dengan **pytest** memberikan cara sederhana namun efektif untuk menjaga kualitas aplikasi Pyramid.
Dengan pengujian rutin di setiap tahap pengembangan, developer dapat memastikan stabilitas dan keandalan kode tanpa harus bergantung pada pengujian manual.
