# Application Configuration dengan .ini Files

## Deskripsi

Tutorial ini menunjukkan cara menggunakan command `pserve` dari Pyramid dengan file konfigurasi `.ini` untuk menjalankan aplikasi dengan lebih sederhana dan lebih baik.

Pyramid memiliki konsep **configuration yang terpisah dari code (first-class concept)**. Pendekatan ini opsional, tetapi membuat Pyramid berbeda dari framework Python lainnya.

### Cara Kerja

Pyramid memanfaatkan library **Setuptools** yang menetapkan konvensi untuk:
- Instalasi Python projects
- Penyediaan "entry points"

**Entry point** digunakan Pyramid untuk mengetahui lokasi WSGI app.

## Objektif

- Menambahkan entry point pada `setup.py` untuk menunjuk lokasi WSGI app  
- Membuat aplikasi berbasis file `.ini`  
- Menjalankan aplikasi menggunakan command `pserve`  
- Memindahkan kode aplikasi ke dalam `__init__.py` package  

## Langkah-langkah

### 1. Copy Project Sebelumnya
```bash
cd ..; cp -r package ini; cd ini
````

### 2. Update File `setup.py`

Tambahkan entry point di dalam fungsi `setup()`:

```python
from setuptools import setup

requires = [
    'pyramid',
    'waitress',
]

setup(
    name='tutorial',
    install_requires=requires,
    entry_points={
        'paste.app_factory': [
            'main = tutorial:main'
        ],
    },
)
```

### 3. Install atau Reinstall Project

```bash
$VENV/bin/pip install -e .
```

### 4. Buat File Konfigurasi `.ini`

Buat file `development.ini` di dalam direktori project:

```ini
[app:main]
use = egg:tutorial

[server:main]
use = egg:waitress#main
listen = localhost:6543
```

### 5. Refactor Startup Code ke `__init__.py`

Pindahkan kode dari `app.py` ke `tutorial/__init__.py`:

```python
from pyramid.config import Configurator
from pyramid.response import Response

def hello_world(request):
    return Response('<body><h1>Hello World!</h1></body>')

def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.add_route('hello', '/')
    config.add_view(hello_world, route_name='hello')
    return config.make_wsgi_app()
```

### 6. Hapus File `app.py` yang Tidak Terpakai

```bash
rm tutorial/app.py
```

### 7. Jalankan Aplikasi

```bash
$VENV/bin/pserve development.ini --reload
```

### 8. Buka Browser

Akses: `http://localhost:6543/`

## Analisis

### Alur Bootstrap Aplikasi

File `development.ini` dibaca oleh `pserve` untuk mem-boot aplikasi:

1. **pserve membaca `[app:main]`** → menemukan `use = egg:tutorial`
2. **Mencari entry point** → `setup.py` mendefinisikan entry point `tutorial:main`
3. **Package `tutorial` memiliki function `main()`** → berada di `__init__.py`
4. **Function main() dipanggil** dengan konfigurasi dari file `.ini`

### Penjelasan `development.ini`

#### `[app:main]`

```ini
[app:main]
use = egg:tutorial
```

Menunjukkan bahwa `pserve` harus menggunakan package `tutorial` sesuai entry point di `setup.py`.

#### `[server:main]`

```ini
[server:main]
use = egg:waitress#main
listen = localhost:6543
```

Menentukan WSGI server (`waitress`) dan port yang digunakan (`6543`).

### Penjelasan Entry Point

```python
entry_points={
    'paste.app_factory': [
        'main = tutorial:main'
    ],
}
```

Format: `nama_entry_point = package:function`

* `main` = nama entry point
* `tutorial` = nama package
* `main` (setelah `:`) = function di `__init__.py`

### Fungsi Lain File `.ini`

Selain untuk bootstrap, `.ini` juga berfungsi untuk:

1. **Konfigurasi WSGI Server**
   Menghubungkan aplikasi ke server dan port.
2. **Konfigurasi Python Logging**
   Mengatur log output di console selama startup dan setiap request.

### Perubahan dari Step Sebelumnya

**Sebelumnya (di `app.py`):**

```python
if __name__ == '__main__':
    with Configurator() as config:
        config.add_route('hello', '/')
        config.add_view(hello_world, route_name='hello')
        app = config.make_wsgi_app()
    serve(app, host='0.0.0.0', port=6543)
```

**Sekarang (di `__init__.py`):**

```python
def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.add_route('hello', '/')
    config.add_view(hello_world, route_name='hello')
    return config.make_wsgi_app()
```

**Tujuan perubahan:**

* Startup code dipindahkan ke `__init__.py` (konvensi umum di Pyramid)
* Bootstrapping WSGI app tidak lagi hardcoded
* Konfigurasi kini dikendalikan oleh `.ini` file

### Command `pserve`

```bash
$VENV/bin/pserve development.ini --reload
```

**Flag `--reload`:**

* Memantau perubahan file (Python, `.ini`, dsb)
* Otomatis me-restart aplikasi
* Sangat membantu saat development

## Output

<img width="1919" height="1018" alt="Screenshot 2025-11-13 001330" src="https://github.com/user-attachments/assets/5b050676-a0ec-48a8-9c0e-083da7632556" />

## Extra Credit

### 1. Apakah bisa tanpa `.ini` configuration?

Ya, bisa. Namun menggunakan `.ini` memberi keuntungan seperti:

* Pemisahan konfigurasi dari kode (separation of concerns)
* Lebih mudah mengelola environment berbeda
* Logging terpusat
* Tidak perlu rebuild/restart untuk mengubah setting

**Alternatif tanpa `.ini`:**

```python
if __name__ == '__main__':
    settings = {
        'pyramid.reload_templates': True,
        'pyramid.debug_all': True,
    }
    config = Configurator(settings=settings)
    app = config.make_wsgi_app()
    serve(app, host='localhost', port=6543)
```

### 2. Apakah bisa punya beberapa `.ini` file?

Bisa — untuk environment berbeda:

```
project/
├── development.ini
├── production.ini
├── staging.ini
└── testing.ini
```

**Contoh perbedaan antar environment:**

```ini
# development.ini
pyramid.debug_all = true
listen = localhost:6543

# production.ini
pyramid.debug_all = false
listen = 0.0.0.0:80
```

### 3. Mengapa entry point tidak menulis `__init__.py`?

Karena Python otomatis mencari file `__init__.py` ketika mengimpor package.

```python
entry_points={
    'paste.app_factory': [
        'main = tutorial:main'
    ],
}
```

Artinya:

* `tutorial` = nama package
* `main` = function di `tutorial/__init__.py`
  Tidak perlu menulis `tutorial.__init__:main`.

### 4. Apa fungsi `**settings` dan arti tanda `**`?

```python
def main(global_config, **settings):
    config = Configurator(settings=settings)
```

`**settings` menangkap semua keyword arguments dalam bentuk dictionary.
Dalam konteks Pyramid, nilainya berasal dari konfigurasi `.ini`.

**Contoh:**

```python
main({}, debug=True, port=6543)
# settings = {'debug': True, 'port': 6543}
```

Dari `.ini`:

```ini
[app:main]
pyramid.reload_templates = true
pyramid.debug_all = true
```

Nilai tersebut otomatis masuk ke `settings`.

Keuntungan:

* Fleksibel untuk menambah konfigurasi
* Dapat diubah per environment tanpa ubah kode
* Clean dan maintainable
