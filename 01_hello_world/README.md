# Single-File Web Applications

## Deskripsi

Tutorial ini menunjukkan cara membuat aplikasi web **Pyramid** paling sederhana menggunakan **single-file module**.

**Microframeworks** adalah framework dengan overhead mental yang rendah mereka melakukan sedikit hal sehingga kamu hanya perlu fokus pada aplikasi sendiri.

**Pyramid** istimewa karena dapat bertindak sebagai **single-file module microframework** sekaligus memiliki kemampuan untuk **scale** ke aplikasi besar.

### Konsep Penting

- **WSGI (Web Server Gateway Interface)**: Standar Python yang mendefinisikan bagaimana aplikasi web Python terhubung ke server, menerima request, dan mengembalikan response.  
- **MVC (Model-View-Controller)**: Pola aplikasi di mana data dalam model memiliki view yang memediasi interaksi dengan sistem luar.

## Objektif

- Menjalankan aplikasi web Pyramid sesederhana mungkin.  
- Memahami dasar **WSGI apps**, **requests**, **views**, dan **responses**.  
- Membuat fondasi yang jelas untuk menambah kompleksitas aplikasi.

## Langkah-langkah

### 1. Setup Directory
```bash
cd ~/projects/quick_tutorial
mkdir hello_world
cd hello_world
````

### 2. Buat File `app.py`

Isi dengan kode berikut:

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

### 3. Jalankan Aplikasi

```bash
$VENV/bin/python app.py
```

### 4. Buka Browser

Akses: `http://localhost:6543/`

## Analisis Kode

### Line 11: `if __name__ == '__main__':`

Bagian ini menandakan titik mulai program ketika dijalankan dari command line, bukan ketika modul ini diimpor oleh file lain.

### Lines 12–14: Configurator

```python
with Configurator() as config:
    config.add_route('hello', '/')
    config.add_view(hello_world, route_name='hello')
```

Menggunakan Pyramid's `Configurator` dalam context manager untuk menghubungkan view code ke URL route tertentu.
`Configurator` berperan sentral dalam pengembangan Pyramid.

### Lines 6–8: View Function

```python
def hello_world(request):
    print('Incoming request')
    return Response('<body><h1>Hello World!</h1></body>')
```

Implementasi view function yang menghasilkan HTTP response. View menerima object `request` dan mengembalikan object `Response`.

### Lines 15–17: WSGI Server

```python
app = config.make_wsgi_app()
serve(app, host='0.0.0.0', port=6543)
```

Mempublikasikan WSGI app menggunakan HTTP server **Waitress**.
`make_wsgi_app()` membuat aplikasi yang kompatibel dengan standar WSGI.

## Konsep Kunci

**Application Configuration** adalah ide sentral dalam Pyramid — membangun aplikasi dari komponen yang loosely-coupled.
Konsep ini akan sering digunakan dalam tutorial berikutnya.

## Output

<img width="1919" height="1015" alt="Screenshot 2025-11-12 230256" src="https://github.com/user-attachments/assets/d07be4bb-46fc-4f94-9ec8-cf355ef2aa5e" />

## Extra Credit

### 1. Mengapa menggunakan `print('Incoming request')` bukan `print 'Incoming request'`?

Karena kode ini ditulis untuk **Python 3**.
Di Python 3, `print` adalah fungsi, sehingga memerlukan tanda kurung `()`.
Di Python 2, `print` adalah statement dan tidak memerlukan kurung.

```python
# Python 3 (benar)
print('Incoming request')

# Python 2 (legacy)
print 'Incoming request'
```

### 2. Apa yang terjadi jika mengembalikan string HTML atau sequence of integers?

**String HTML:**

```python
def hello_world(request):
    return '<h1>Hello</h1>'  # Error!
```

Akan menghasilkan **error** karena Pyramid view harus mengembalikan object `Response`, bukan string biasa.

**Sequence of Integers:**

```python
def hello_world(request):
    return [1, 2, 3]  # Error!
```

Juga akan menghasilkan **error**, karena nilai kembalian harus berupa `Response` object atau sesuatu yang kompatibel dengan WSGI.

**Solusi yang benar:**

```python
return Response('<h1>Hello</h1>')
```

### 3. Apa yang terjadi jika ada kode invalid seperti `print xyz`?

Ketika menambahkan kode:

```python
def hello_world(request):
    print xyz  # Invalid!
    return Response('<body><h1>Hello World!</h1></body>')
```

Lalu menjalankan ulang server dan me-*reload* browser:

* **Exception muncul di console** tempat `python app.py` dijalankan
* Browser menampilkan error page atau timeout
* Console menunjukkan traceback lengkap, misalnya `NameError: name 'xyz' is not defined`

Ini sangat membantu untuk debugging karena error dapat dilihat langsung.

### 4. GI dalam WSGI adalah “Gateway Interface” — dimodelkan dari standar web apa?

Jawaban: **CGI (Common Gateway Interface)**

**Perbedaannya:**

* **CGI**: Menjalankan proses baru untuk setiap request (lambat)
* **WSGI**: Aplikasi tetap berjalan dan hanya memanggil fungsi Python (lebih cepat dan efisien)

## Dependencies

Pastikan sudah menginstal:

```bash
pip install pyramid waitress
```

## Port Default

Aplikasi berjalan di:

* `http://localhost:6543`
* Atau melalui IP komputer dalam jaringan (jika firewall mengizinkan).
