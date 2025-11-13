# Python Packages untuk Aplikasi Pyramid

## Deskripsi

Tutorial ini menjelaskan cara mengorganisir aplikasi **"Hello World"** menjadi Python package yang lebih terstruktur menggunakan **`setup.py`**.

Python package adalah cara untuk mengelompokkan kumpulan modul dan file menjadi satu unit yang terorganisir dengan namespace sendiri. Pendekatan ini adalah praktik umum dalam pengembangan Python modern dan digunakan secara efektif oleh Pyramid.

### Apa itu Python Package?

- **Package** = direktori yang memiliki file khusus bernama `__init__.py`
- Jika direktori ada di `sys.path` dan memiliki `__init__.py`, maka Python akan mengenalinya sebagai package
- Package bisa dibundle, diinstall, dan didistribusikan menggunakan **`setup.py`**

### Struktur Pengembangan

Struktur project pada tahap ini mengikuti pola berikut:

1. **Project directory** → folder utama untuk tiap langkah tutorial  
2. **setup.py** → file yang mengatur konfigurasi dan dependensi project  
3. **tutorial subdirectory** → dijadikan Python package dengan menambahkan `__init__.py`  
4. **Development mode** → project diinstall menggunakan `pip install -e .` agar mudah dikembangkan  

> Pengembangan dilakukan di dalam **Python package**, dan package tersebut merupakan bagian dari sebuah **Python project**.

## Objektif

- Membuat direktori Python *package* dengan `__init__.py`  
- Menyiapkan *project* minimal menggunakan `setup.py`  
- Menginstall project dalam *development mode*

## Langkah-langkah

### 1. Buat Directory untuk Tutorial
```bash
cd .. 
mkdir package 
cd package
````

### 2. Buat File `setup.py`

```python
from setuptools import setup

# List dependencies yang akan diinstall otomatis lewat `pip install -e .`
requires = [
    'pyramid',
    'waitress',
]

setup(
    name='tutorial',
    install_requires=requires,
)
```

### 3. Install Project dalam Development Mode

```bash
$VENV/bin/pip install -e .
mkdir tutorial
```

### 4. Buat File `__init__.py`

```python
# package
```

### 5. Buat File Aplikasi `tutorial/app.py`

```python
from waitress import serve
from pyramid.config import Configurator
from pyramid.response import Response

def hello_world(request):
    print('Incoming request')
    return Response('<body><h1>Hello World!</h1></body>')

if __name__ == '__main__':
    with Configurator() as config:
        config.add_route('hello', '/')
        config.add_view(hello_world, route_name='hello')
        app = config.make_wsgi_app()
    serve(app, host='0.0.0.0', port=6543)
```

### 6. Jalankan Aplikasi

```bash
$VENV/bin/python tutorial/app.py
```

### 7. Buka Browser

Akses: `http://localhost:6543/`

## Analisis

### Python Package vs Python Project

**Python Package:**

* Memberikan unit pengembangan yang terorganisir
* Menggunakan direktori dengan file `__init__.py`

**Python Project:**

* Dikelola melalui file `setup.py`
* Memungkinkan pengaturan dependencies dan metadata project
* Dapat diinstall dalam *editable mode* (`pip install -e .`), sehingga setiap perubahan kode langsung terdeteksi tanpa reinstall

### Penjelasan setup.py

```python
requires = [
    'pyramid',
    'waitress',
]
```

Daftar dependencies yang akan diinstall otomatis ketika menjalankan `pip install -e .`

```python
setup(
    name='tutorial',
    install_requires=requires,
)
```

Konfigurasi minimal untuk project dengan nama `tutorial` dan dependencies yang diperlukan agar aplikasi dapat berjalan.

### Nama Package: `tutorial`

Nama **tutorial** digunakan secara konsisten di setiap langkah untuk menghindari kebingungan dan menjaga keseragaman struktur project.

### Catatan

Menjalankan aplikasi menggunakan:

```bash
python tutorial/app.py
```

adalah pendekatan sementara untuk tujuan pembelajaran. Dalam proyek nyata, biasanya aplikasi dijalankan melalui entry point atau server configuration yang lebih terintegrasi.

## Output

<img width="1919" height="1011" alt="Screenshot 2025-11-12 232620" src="https://github.com/user-attachments/assets/c1673b27-08d5-47b8-b146-f34c2c196d34" />

## Perbandingan dengan Single-File

| Aspek        | Single-File     | Python Package                 |
| ------------ | --------------- | ------------------------------ |
| Struktur     | 1 file `app.py` | Directory dengan `__init__.py` |
| Dependencies | Manual install  | Otomatis via `setup.py`        |
| Skalabilitas | Terbatas        | Mudah dikembangkan             |
| Organisasi   | Minimal         | Terstruktur                    |
| Distribution | Sulit           | Mudah di-share                 |

## Keuntungan Menggunakan Package

1. **Organisasi lebih baik** – File terpisah sesuai fungsi
2. **Dependency management** – Dependencies tercatat di `setup.py`
3. **Editable mode** – Perubahan langsung aktif tanpa reinstall
4. **Scalability** – Mudah menambah modul dan fitur baru
5. **Distribution** – Dapat di-package dan dibagikan ke developer lain dengan mudah
