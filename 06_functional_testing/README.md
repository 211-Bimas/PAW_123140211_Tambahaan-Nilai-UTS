# Functional Testing dengan WebTest di Pyramid

## Pendahuluan

Unit testing berguna untuk menguji bagian-bagian kecil dari aplikasi secara terpisah. Tapi dalam aplikasi web, kita juga perlu memastikan bahwa tampilan (template) dan keseluruhan sistem berjalan dengan baik.
Di sinilah **functional testing** digunakan yaitu cara menguji aplikasi web secara penuh dari awal sampai akhir.

**WebTest** adalah library Python untuk melakukan functional testing. Dengan WebTest, kita bisa mensimulasikan permintaan HTTP (seperti yang dilakukan browser) langsung ke aplikasi WSGI.
Kelebihannya, WebTest tidak perlu menyalakan server HTTP sungguhan, jadi pengujiannya tetap cepat dan cocok untuk TDD (Test-Driven Development).

## Perbedaan Unit Testing vs Functional Testing

| Unit Testing                                    | Functional Testing                               |
| ----------------------------------------------- | ------------------------------------------------ |
| Menguji bagian kecil seperti fungsi atau method | Menguji aplikasi web secara keseluruhan          |
| Menggunakan request palsu (`DummyRequest`)      | Mensimulasikan HTTP request sungguhan            |
| Lebih cepat dan terisolasi                      | Sedikit lebih lambat tapi hasilnya lebih lengkap |
| Tidak menguji template atau tampilan            | Menguji semua bagian termasuk template           |

## Setup Project

### 1. Instalasi WebTest

Tambahkan `webtest` ke dalam dependency development di file `setup.py`.

### 2. Install Dependencies

```bash
$VENV/bin/pip install -e ".[dev]"
```

## Menulis Functional Test

Tambahkan atau ubah file `tutorial/tests.py` untuk menambahkan functional test.

### Menjalankan Test

```bash
$VENV/bin/pytest tutorial/tests.py -q
```

Output: 

<img width="1231" height="446" alt="Screenshot 2025-11-13 204434" src="https://github.com/user-attachments/assets/62063ccf-5b9e-4947-8b65-28e9544f85ac" />

## ANALISIS

### Struktur Functional Test Class

```python
class TutorialFunctionalTests(unittest.TestCase):
```

Class ini dipisahkan dari unit test supaya kode lebih rapi dan mudah dipelihara. Functional test biasanya ditulis di class khusus agar tidak bercampur dengan unit test.

### setUp Method

```python
def setUp(self):
    from tutorial import main
    app = main({})
    from webtest import TestApp

    self.testapp = TestApp(app)
```

**Penjelasan:**

1. **`from tutorial import main`** → Mengimpor fungsi utama aplikasi Pyramid.
2. **`app = main({})`** → Membuat instance aplikasi dengan konfigurasi kosong.
3. **`from webtest import TestApp`** → Mengimpor `TestApp` dari WebTest.
4. **`self.testapp = TestApp(app)`** → Membungkus aplikasi Pyramid agar bisa dites dengan simulasi HTTP request.

`TestApp` berfungsi seperti browser mini — dia bisa mengirim request dan membaca response tanpa benar-benar menjalankan server web.

### Test Method

```python
def test_hello_world(self):
    res = self.testapp.get('/', status=200)
    self.assertIn(b'<h1>Hello World!</h1>', res.body)
```

**Penjelasan:**

1. **`self.testapp.get('/', status=200)`**

   * Mengirim HTTP GET ke path `'/'`.
   * Memastikan status code adalah 200 (berhasil).
   * Mengembalikan objek response.

2. **`self.assertIn(b'<h1>Hello World!</h1>', res.body)`**

   * Mengecek apakah HTML `<h1>Hello World!</h1>` muncul dalam response.
   * `res.body` berisi konten HTML dalam bentuk *bytes*.
   * `b''` menandakan bahwa data yang dibandingkan adalah *byte string* (lihat bagian Extra Credit).

### End-to-End Testing

Functional test ini disebut **end-to-end** karena:

1. Menjalankan aplikasi sebenarnya dengan `main({})`.
2. Mensimulasikan HTTP request sungguhan.
3. Menguji seluruh alur dari routing → view → template → response.
4. Memastikan hasil HTML sesuai harapan.

Berbeda dengan unit test yang hanya menguji satu fungsi, functional test memastikan semua komponen bekerja bersama dengan baik.

### Kecepatan Eksekusi

Meskipun functional test lebih lengkap, WebTest tetap cepat karena:

* Tidak perlu menjalankan server HTTP sungguhan.
* Menggunakan pemanggilan langsung ke aplikasi WSGI.
* Hanya butuh waktu sangat singkat (sekitar 0.25 detik untuk 2 test).

---

## EXTRA CREDIT

### 1. Mengapa Functional Test Menggunakan `b''`?

**Pertanyaan:**
Kenapa dalam functional test kita memakai `b''`?

**Contoh:**

```python
self.assertIn(b'<h1>Hello World!</h1>', res.body)
```

**Penjelasan:**

#### Alasan Teknis

1. **`res.body` adalah bytes, bukan string.**
   Di Python 3, hasil response HTTP disimpan dalam bentuk *bytes* (data mentah).

2. **Tipe data harus sama ketika dibandingkan.**
   String (`'...'`) tidak bisa langsung dibandingkan dengan bytes (`b'...'`).

#### Contoh

```python
# BENAR (bytes dibandingkan bytes)
self.assertIn(b'<h1>Hello World!</h1>', res.body)

# SALAH (string dibandingkan bytes)
self.assertIn('<h1>Hello World!</h1>', res.body)
```

#### Alternatif: Gunakan `res.text`

Jika ingin membandingkan teks biasa, bisa gunakan `res.text` yang otomatis mengubah bytes jadi string.

```python
res = self.testapp.get('/', status=200)
self.assertIn('<h1>Hello World!</h1>', res.text)
```

#### Kapan Menggunakan Masing-Masing

| Gunakan `res.body` (bytes)               | Gunakan `res.text` (string)             |
| ---------------------------------------- | --------------------------------------- |
| Saat bekerja dengan data mentah (binary) | Saat bekerja dengan teks atau HTML      |
| Ketika butuh hasil asli dari response    | Ketika ingin membandingkan string biasa |
| Lebih sesuai dengan standar HTTP         | Lebih mudah dibaca oleh developer       |

---

## Keuntungan Functional Testing dengan WebTest

1. **End-to-End Coverage** – Menguji seluruh alur aplikasi, bukan hanya fungsi tunggal.
2. **Template Testing** – Memastikan tampilan (HTML) dirender dengan benar.
3. **Fast Execution** – Tidak perlu server sungguhan, test tetap cepat.
4. **Terintegrasi dengan pytest** – Semua hasil test muncul bersama unit test.
5. **Realistic Testing** – Meniru interaksi pengguna sebenarnya.
6. **Mudah Ditulis** – Sintaks WebTest sederhana dan mudah dipahami.

---

## Kesimpulan

Functional testing dengan WebTest melengkapi unit testing dengan memberikan pengujian menyeluruh terhadap aplikasi web.
Gabungan antara **unit test** (cepat dan fokus) dengan **functional test** (lengkap dan realistis) membuat kualitas aplikasi lebih terjamin.

Dengan WebTest, kita bisa:

* Mensimulasikan interaksi pengguna sebenarnya
* Mengecek hasil HTML dari template
* Menjalankan test cepat dan efisien
* Menulis test yang mudah dibaca dan dipelihara
