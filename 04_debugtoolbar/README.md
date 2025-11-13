# Easier Development dengan Debug Toolbar

## Deskripsi

Tutorial ini menjelaskan cara menggunakan add-on **`pyramid_debugtoolbar`** untuk menangani error dan melakukan introspeksi selama tahap pengembangan.  
Toolbar ini menampilkan berbagai alat debugging langsung di browser sehingga mempercepat proses analisis dan perbaikan masalah.

Pada tahap sebelumnya, kita sudah membahas tentang template reloading dan opsi `--reload` untuk reloading aplikasi.  
Kini, kita akan menambahkan alat debugging yang lebih powerful dan interaktif.

### Apa itu pyramid_debugtoolbar?

**`pyramid_debugtoolbar`** adalah add-on populer di Pyramid yang menyediakan fitur seperti:
- Monitoring performa dan profiling request
- Inspeksi request dan response
- Traceback interaktif untuk debugging error
- Informasi rendering template dan konfigurasi
- Akses introspektif terhadap komponen aplikasi

Menggunakan add-on ini memperlihatkan bagaimana Pyramid mengonfigurasi dan mengintegrasikan komponen eksternal dengan fleksibel.

## Objektif

- Menginstal dan mengaktifkan toolbar untuk membantu proses development  
- Memahami konsep **Pyramid add-ons**  
- Menunjukkan bagaimana add-on dikonfigurasi dan diintegrasikan ke aplikasi

## Langkah-langkah

### 1. Copy Project Sebelumnya
```bash
cd ..; cp -r ini debugtoolbar; cd debugtoolbar
````

### 2. Update File `setup.py`

Tambahkan dependency untuk pengembangan (`dev_requires`) di bawah `extras_require`:

```python
from setuptools import setup

# Dependencies utama
requires = [
    'pyramid',
    'waitress',
]

# Dependencies khusus untuk development
dev_requires = [
    'pyramid_debugtoolbar',
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

### 3. Install Project dengan Development Extras

Gunakan `pip` untuk menginstall dependencies tambahan:

```bash
$VENV/bin/pip install -e ".[dev]"
```

`[dev]` menandakan bahwa kita juga ingin menginstal paket tambahan yang didefinisikan di `extras_require`.

### 4. Update File `development.ini`

Tambahkan baris `pyramid.includes` untuk memuat add-on `pyramid_debugtoolbar`:

```ini
[app:main]
use = egg:tutorial
pyramid.includes =
    pyramid_debugtoolbar

[server:main]
use = egg:waitress#main
listen = localhost:6543
```

### 5. Jalankan Aplikasi

```bash
$VENV/bin/pserve development.ini --reload
```

### 6. Buka Browser

Akses: `http://localhost:6543/`
Toolbar akan muncul di sisi kanan layar.

---

## Analisis

### Apa itu Pyramid Add-on?

`pyramid_debugtoolbar` merupakan **Python package biasa** (tersedia di PyPI), sekaligus **Pyramid add-on** yang perlu di-*include* ke aplikasi agar konfigurasinya dijalankan.
Ada dua cara untuk melakukan *include* add-on:

#### 1. Melalui Imperative Configuration (di kode Python)

```python
# tutorial/__init__.py
def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.include('pyramid_debugtoolbar')  # include secara langsung
    return config.make_wsgi_app()
```

#### 2. Melalui Declarative Configuration (di file .ini)

```ini
[app:main]
pyramid.includes =
    pyramid_debugtoolbar
```

Tutorial ini menggunakan cara kedua karena lebih bersih dan mudah dimatikan tanpa mengubah kode.

---

### Bagaimana Cara Kerjanya

1. `pserve` membaca `development.ini`
2. Menemukan `pyramid.includes = pyramid_debugtoolbar`
3. Pyramid otomatis memanggil fungsi konfigurasi milik `pyramid_debugtoolbar`
4. Toolbar menyisipkan HTML/CSS kecil sebelum tag `</body>` agar muncul di browser
5. Jika terjadi error, traceback ditampilkan dengan tampilan interaktif

---

### Menonaktifkan Toolbar

Jika ingin menonaktifkannya sementara, cukup komentari baris pada `pyramid.includes`:

```ini
# development.ini
[app:main]
pyramid.includes =
#    pyramid_debugtoolbar
```

Hal ini berguna saat kamu ingin menguji tampilan front-end tanpa modifikasi HTML dari toolbar, atau sebelum deploy ke production.

---

### Konsep Setuptools Extras

Pyramid menggunakan fitur **Setuptools extras** untuk membedakan antara dependencies utama dan opsional.

#### Dalam `setup.py`

```python
extras_require={
    'dev': ['pyramid_debugtoolbar'],
}
```

#### Cara Install

```bash
# Install dengan dev dependencies
pip install -e ".[dev]"
```

#### Tujuan Pemisahan Dependencies

| Jenis              | Tujuan                                                | Contoh                           |
| ------------------ | ----------------------------------------------------- | -------------------------------- |
| `install_requires` | Dependency wajib agar aplikasi berjalan               | `pyramid`, `waitress`            |
| `extras_require`   | Dependency opsional (misal untuk development/testing) | `pyramid_debugtoolbar`, `pytest` |

#### Keuntungan:

1. **Keamanan:** toolbar menampilkan data sensitif, jadi tidak aman di production
2. **Kinerja:** menambah overhead, sebaiknya hanya untuk pengembangan
3. **Kebersihan environment:** deployment production lebih kecil dan cepat
4. **Keteraturan:** memisahkan tools development dari kebutuhan runtime

---

### Contoh Lengkap `setup.py`

```python
setup(
    name='tutorial',
    install_requires=[
        'pyramid',
        'waitress',
    ],
    extras_require={
        'dev': [
            'pyramid_debugtoolbar',
            'pyramid_ipython',
        ],
        'test': [
            'pytest',
            'webtest',
        ],
    },
    entry_points={
        'paste.app_factory': [
            'main = tutorial:main'
        ],
    },
)
```

---

## Output

<img width="1919" height="1019" alt="Screenshot 2025-11-13 202113" src="https://github.com/user-attachments/assets/2ca912ff-d28e-48ab-89a0-2969b6ea63cc" />

---

## Extra Credit

### 1. Mengapa `pyramid_debugtoolbar` ditambahkan di `dev_requires`, bukan `requires`?

Karena `pyramid_debugtoolbar` **hanya digunakan saat development**, bukan saat aplikasi berjalan di production.

* **requires:** berisi dependency inti untuk menjalankan aplikasi
* **dev_requires:** berisi dependency tambahan untuk debugging dan development

**Manfaat pemisahan:**

* Mencegah bocornya data sensitif di production
* Menjaga performa tetap optimal
* Mengurangi ukuran environment production
* Memastikan hanya tools yang diperlukan yang diinstal

---

### 2. Eksperimen: Memunculkan Error dan Menjelajahi Debugger

#### a. Perkenalkan Bug

```python
def hello_world(request):
    return xResponse('<body><h1>Hello World!</h1></body>')
```

Error terjadi karena `xResponse` tidak didefinisikan.

#### b. Jalankan Aplikasi dan Akses `http://localhost:6543/`

Kamu akan melihat tampilan traceback interaktif seperti ini:

```
NameError: name 'xResponse' is not defined
```

#### c. Gunakan Interactive Console

Klik ikon “screen” di bagian bawah traceback untuk membuka Python console di context error.

**Contoh eksplorasi:**

```python
>>> request
<Request GET http://localhost:6543/>

>>> Response
<class 'pyramid.response.Response'>

>>> request.method
'GET'

>>> request.url
'http://localhost:6543/'
```

Kamu juga bisa mencoba:

```python
>>> response = Response('<h1>Test</h1>')
>>> response.status
'200 OK'
>>> response.text
'<h1>Test</h1>'
```

---

### 3. Fitur Lain Toolbar

| Fitur                    | Deskripsi                                              |
| ------------------------ | ------------------------------------------------------ |
| **Traceback Interaktif** | Menelusuri error, frame, dan variabel lokal            |
| **Request Panel**        | Lihat detail request, headers, cookies, session        |
| **Performance Panel**    | Monitor waktu eksekusi dan resource                    |
| **Template Panel**       | Tampilkan variabel rendering dan waktu render          |
| **SQL Panel (opsional)** | Profiling query database (jika menggunakan SQLAlchemy) |

---

### 4. Tips Debugging Efektif

1. Buat error atau exception di aplikasi
2. Biarkan toolbar menampilkan traceback
3. Klik ikon console → eksplorasi variabel
4. Perbaiki kode
5. Simpan → aplikasi otomatis reload (`--reload`)
6. Uji ulang di browser

**Console Tips:**

* Gunakan `dir(obj)` untuk melihat atribut
* Gunakan `help(obj)` untuk dokumentasi
* Tekan `Tab` untuk autocomplete
* Gunakan `↑` / `↓` untuk history

---

## Kesimpulan

Integrasi **`pyramid_debugtoolbar`** memberi:

* Visual feedback langsung saat debugging
* Konsol interaktif berbasis konteks error
* Kemudahan aktivasi tanpa ubah kode
* Praktik dependency management yang bersih dengan `extras_require`

Dengan konfigurasi ini, proses pengembangan menjadi jauh lebih produktif, aman, dan efisien.
