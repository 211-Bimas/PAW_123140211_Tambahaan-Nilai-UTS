# Menyalurkan URL ke Views dengan Routing

## Deskripsi

Tutorial ini menjelaskan cara kerja **routing** di Pyramid Framework untuk mencocokkan pola URL dengan fungsi atau kelas view. Routing memungkinkan aplikasi mengenali bagian dari URL sebagai data dinamis yang dapat digunakan di dalam kode.

Dalam pengembangan web modern, URL tidak hanya berfungsi sebagai alamat halaman, tetapi juga sebagai **sumber data** yang dapat dikirim ke server. Contoh penggunaan umum:

* `/users/42` – menampilkan profil pengguna dengan ID 42
* `/blog/2025/11/artikel-routing` – menampilkan artikel blog tertentu
* `/products/laptop/macbook-pro` – menampilkan detail produk berdasarkan kategori dan nama

### Mengapa Routing Dipisahkan dari View?

Pyramid secara **sengaja memisahkan** konfigurasi route dan view untuk memberikan:

* **Kendali penuh terhadap urutan route** (explicit ordering)
* **Fleksibilitas tinggi** dalam mengatur URL
* **Kemudahan testing dan pemeliharaan** karena struktur yang terpisah dan jelas

Dengan pendekatan ini, developer dapat menyesuaikan desain URL tanpa harus mengubah logika view.

## Tujuan

1. Membuat route dengan pola URL yang berisi parameter dinamis.
2. Mengambil data dari URL melalui `matchdict`.
3. Menggunakan data tersebut di dalam view.
4. Menguji fungsi routing menggunakan unit dan functional tests.

---

## Langkah-langkah Implementasi

### 1. Setup Proyek

```bash
cd ..
cp -r view_classes routing
cd routing
$VENV/bin/pip install -e .
```

---

### 2. Menambahkan Route dengan Replacement Pattern

Edit file `routing/tutorial/__init__.py`

```python
config.add_route('home', '/howdy/{first}/{last}')
```

**Penjelasan:**

* `/howdy/` → bagian statis dari URL
* `{first}` dan `{last}` → variabel yang nilainya diambil dari URL

**Contoh hasil pencocokan:**

| URL                | `first` | `last`  |
| ------------------ | ------- | ------- |
| `/howdy/jane/doe`  | `jane`  | `doe`   |
| `/howdy/amy/smith` | `amy`   | `smith` |

---

### 3. Membuat View

Edit file `routing/tutorial/views.py`

```python
from pyramid.view import view_config, view_defaults

@view_defaults(renderer='home.pt')
class TutorialViews:
    def __init__(self, request):
        self.request = request

    @view_config(route_name='home')
    def home(self):
        first = self.request.matchdict['first']
        last = self.request.matchdict['last']
        return {
            'name': 'Home View',
            'first': first,
            'last': last
        }
```

**Penjelasan:**

* `self.request.matchdict` menyimpan data hasil ekstraksi dari URL.
* Nilai `first` dan `last` diambil dari nama parameter dalam `{}`.
* Dictionary yang dikembalikan akan diteruskan ke template untuk dirender.

---

### 4. Membuat Template

Buat file `routing/tutorial/home.pt`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Quick Tutorial: ${name}</title>
</head>
<body>
    <h1>${name}</h1>
    <p>First: ${first}, Last: ${last}</p>
</body>
</html>
```

**Penjelasan:**
Template menggunakan variabel `${first}` dan `${last}` dari dictionary yang dikembalikan view.

---

### 5. Membuat Tests

Edit file `routing/tutorial/tests.py`

#### Unit Test

```python
request = testing.DummyRequest()
request.matchdict['first'] = 'First'
request.matchdict['last'] = 'Last'
inst = TutorialViews(request)
response = inst.home()
self.assertEqual(response['first'], 'First')
self.assertEqual(response['last'], 'Last')
```

* Menggunakan `DummyRequest()` untuk mensimulasikan request.
* `matchdict` diisi manual untuk meniru data dari URL.

#### Functional Test

```python
res = self.testapp.get('/howdy/Jane/Doe', status=200)
self.assertIn(b'Jane', res.body)
self.assertIn(b'Doe', res.body)
```

* Melakukan request nyata ke aplikasi.
* Memastikan routing dan view bekerja dari ujung ke ujung.

---

### 6. Menjalankan Tests

```bash
$VENV/bin/pytest tutorial/tests.py -q
```

Jika berhasil, hasilnya:

```
..
2 passed in 0.39 seconds
```

Hasil:
<img width="1241" height="448" alt="Screenshot 2025-11-13 233104" src="https://github.com/user-attachments/assets/b16411ad-e4c9-4b0c-8900-893f8ccfe7b5" />

---

### 7. Menjalankan Aplikasi

```bash
$VENV/bin/pserve development.ini --reload
```

**Coba akses:**

* [http://localhost:6543/howdy/amy/smith](http://localhost:6543/howdy/amy/smith)
* [http://localhost:6543/howdy/john/doe](http://localhost:6543/howdy/john/doe)

**Tampilan di browser:**

```
First: amy, Last: smith
```

Hasil:
<img width="1919" height="1020" alt="Screenshot 2025-11-13 233135" src="https://github.com/user-attachments/assets/ebb44e80-9397-4a10-9cc2-85a50d182da7" />

---

## Analisis

### Cara Kerja Routing

1. **User mengakses URL:** `/howdy/amy/smith`
2. **Pyramid Router mencari route yang cocok:**
   `/howdy/{first}/{last}`
3. **Nilai dari URL diambil dan dimasukkan ke `matchdict`:**

   ```python
   request.matchdict = {'first': 'amy', 'last': 'smith'}
   ```
4. **View memanfaatkan data tersebut:**

   ```python
   first = self.request.matchdict['first']
   last = self.request.matchdict['last']
   ```
5. **Data diteruskan ke template untuk ditampilkan.**

---

### Mengenal `matchdict`

`request.matchdict` adalah dictionary yang berisi nilai dari pola dinamis di URL.

Contoh:

```python
# Route: /blog/{year}/{month}/{slug}
# URL: /blog/2025/11/routing-pyramid

request.matchdict = {
    'year': '2025',
    'month': '11',
    'slug': 'routing-pyramid'
}
```

---

### Jenis Replacement Pattern

1. **Sederhana**

   ```python
   config.add_route('user', '/users/{id}')
   ```

   Mencocokkan `/users/123`

2. **Beberapa Parameter**

   ```python
   config.add_route('blog', '/blog/{year}/{month}/{slug}')
   ```

3. **Dengan Regex**

   ```python
   config.add_route('user', '/users/{id:\d+}')
   ```

   Hanya cocok untuk angka (`\d+`)

4. **Catch-All**

   ```python
   config.add_route('files', '/files/*subpath')
   ```

   Cocok untuk banyak segmen sekaligus

---

## Extra Credit: Mengakses `/howdy` Tanpa Parameter

Jika mengakses:

```
http://localhost:6543/howdy
```

**Hasil:**
`404 Not Found`

**Alasannya:**
Pyramid mengharuskan URL memenuhi semua pola pada route:

```python
config.add_route('home', '/howdy/{first}/{last}')
```

Tanpa `first` dan `last`, route tidak cocok, sehingga menghasilkan 404.

**Solusi:**
Menambahkan route alternatif:

```python
config.add_route('home_default', '/howdy')
```

dan menyesuaikan view agar bisa menangani default value:

```python
first = self.request.matchdict.get('first', 'Guest')
last = self.request.matchdict.get('last', '')
```

---

## Konsep Penting

### 1. Urutan Route (Route Ordering)

Urutan pendaftaran route menentukan prioritas pencocokan:

```python
config.add_route('specific', '/users/admin')
config.add_route('general', '/users/{id}')
```

URL `/users/admin` akan cocok dengan `specific` karena didefinisikan lebih dulu.

### 2. Perbedaan `matchdict` vs `params`

| Atribut     | Sumber Data  | Contoh URL      | Cara Akses                |
| ----------- | ------------ | --------------- | ------------------------- |
| `matchdict` | Path URL     | `/users/123`    | `request.matchdict['id']` |
| `params`    | Query string | `/users?id=123` | `request.params['id']`    |

### 3. Nama Route (Route Names)

Nama route memudahkan pembuatan URL secara dinamis:

```python
request.route_url('home', first='john', last='doe')
# Menghasilkan: /howdy/john/doe
```

---

## Kesimpulan

Routing adalah bagian penting dari arsitektur web. Di Pyramid, sistem routing menyediakan:

* **Kejelasan dan eksplisitasi** dalam pencocokan URL
* **Fleksibilitas tinggi** dalam mendefinisikan pola URL
* **Kemudahan pengujian dan pemeliharaan**
* **Integrasi kuat** dengan objek `request` untuk akses data dari URL

Dengan memahami konsep routing, developer dapat membuat struktur URL yang rapi, terukur, dan mudah dikelola di seluruh aplikasi web.
