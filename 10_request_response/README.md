# Menangani Web Requests dan Responses

## Deskripsi

Tutorial ini membahas cara kerja Pyramid Framework dalam menangani **permintaan (request)** dan **tanggapan (response)** pada aplikasi web.
Setiap aplikasi web bekerja dengan menerima permintaan dari pengguna (misalnya melalui browser) dan mengirimkan tanggapan kembali. Proses ini sangat penting karena menjadi inti dari semua komunikasi di web.

Pyramid menggunakan library **WebOb**, yang sudah mapan dan banyak digunakan di dunia Python, untuk menangani dua hal utama:

* **Request:** membaca data dari URL, parameter, header, atau cookies.
* **Response:** membuat tanggapan yang akan dikirim kembali ke pengguna, lengkap dengan isi (body), status kode, dan header.

## Tujuan

1. Memahami bagaimana Pyramid menangani request dan response.
2. Mempelajari cara mengambil data dari request (misalnya dari URL parameter).
3. Mengetahui cara mengatur isi dan informasi pada response.
4. Mempelajari cara membuat redirect dengan benar menggunakan HTTP response khusus.

## Langkah-langkah Implementasi

### 1. Setup Proyek

```bash
cd ..
cp -r view_classes request_response
cd request_response
$VENV/bin/pip install -e .
```

### 2. Konfigurasi Routes

Edit file `request_response/tutorial/__init__.py`.

**Penjelasan:**

* `home` (`/`) → halaman utama, berfungsi untuk melakukan redirect.
* `plain` (`/plain`) → halaman tujuan yang menampilkan teks sederhana.

### 3. Membuat Views

Edit file `request_response/tutorial/views.py`.

#### Penjelasan Kode

**View `home()` – Melakukan Redirect**

```python
@view_config(route_name='home')
def home(self):
    return HTTPFound(location='/plain')
```

* `HTTPFound` adalah jenis response khusus dengan status kode **302 (redirect)**.
* Saat pengguna membuka `/`, browser otomatis diarahkan ke `/plain`.

**View `plain()` – Membaca Request dan Mengirim Response**

```python
@view_config(route_name='plain')
def plain(self):
    name = self.request.params.get('name', 'No Name Provided')
    body = 'URL %s with name: %s' % (self.request.url, name)
    return Response(
        content_type='text/plain',
        body=body
    )
```

**Penjelasan:**

* `self.request.params` digunakan untuk membaca data dari URL.
* Jika parameter `name` tidak dikirim, maka akan muncul teks *“No Name Provided”*.
* `Response()` digunakan untuk membuat tanggapan dalam format teks biasa.

### 4. Mengubah dan Menjalankan Test

Edit file `request_response/tutorial/tests.py`.

**Contoh Unit Test:**

```python
def test_home(self):
    response = inst.home()
    self.assertEqual(response.status, '302 Found')
```

→ Memastikan bahwa view `home()` benar-benar melakukan redirect.

```python
request = testing.DummyRequest()
request.GET['name'] = 'Jane Doe'
response = inst.plain()
self.assertIn(b'Jane Doe', response.body)
```

→ Menguji bahwa parameter dari URL terbaca dengan benar.

**Functional Test:**

```python
res = self.testapp.get('/plain?name=Jane%20Doe', status=200)
```

→ Menjalankan simulasi akses web nyata menggunakan *WebTest* untuk memastikan hasil akhir sesuai harapan.

### 5. Menjalankan Tests

```bash
$VENV/bin/pytest tutorial/tests.py -q
```

Jika berhasil, hasilnya akan seperti ini:

```
....
5 passed in 0.30 seconds
```

Hasil :
<img width="1238" height="446" alt="Screenshot 2025-11-13 231852" src="https://github.com/user-attachments/assets/c8ed79b1-4111-43e9-bd13-a64527d80697" />


---

### 6. Menjalankan Aplikasi

```bash
$VENV/bin/pserve development.ini --reload
```

**Akses di Browser:**

* `http://localhost:6543/` → akan otomatis diarahkan ke `/plain`
* `http://localhost:6543/plain` → menampilkan teks *“No Name Provided”*
* `http://localhost:6543/plain?name=alice` → menampilkan teks *“URL ... with name: alice”*

**Hasil:** 
<img width="1919" height="1017" alt="Screenshot 2025-11-13 232435" src="https://github.com/user-attachments/assets/8b13cf59-c958-40fe-809f-767e1d01c434" />

<img width="1919" height="1018" alt="Screenshot 2025-11-13 232445" src="https://github.com/user-attachments/assets/f7a859ba-77a7-4060-b3b8-b84ba8f42317" />

<img width="1919" height="1014" alt="Screenshot 2025-11-13 232456" src="https://github.com/user-attachments/assets/f15538ed-ee2c-4af2-aa7f-d9f469c29db3" />

---

## Analisis

### Alur Kerja Aplikasi

1. **Pengguna membuka `/`**

   * Fungsi `home()` dijalankan.
   * Mengembalikan response `HTTPFound` (redirect ke `/plain`).

2. **Browser diarahkan ke `/plain`**

   * Fungsi `plain()` dijalankan.
   * Membaca parameter `name` dari URL.
   * Membuat response teks biasa dengan informasi URL dan nama.

3. **Pengguna membuka `/plain?name=alice`**

   * Parameter `name=alice` terbaca.
   * Response menampilkan teks yang berisi nama tersebut.

### Request Object

`Request` berisi semua informasi yang dikirim pengguna ke server.

Contoh data yang bisa diakses:

```python
self.request.url         # URL lengkap
self.request.path        # Hanya path (/plain)
self.request.GET         # Parameter di URL (?name=...)
self.request.headers     # Header HTTP
self.request.cookies     # Cookies
self.request.json_body   # Data JSON (jika ada)
```

### Response Object

`Response` berisi semua informasi yang dikirim server ke pengguna.

Contoh:

```python
Response(
    body='Hello World',
    status='200 OK',
    content_type='text/plain'
)
```

Bisa juga menambahkan headers atau cookies sesuai kebutuhan.

### Redirect (Pengalihan Halaman)

Ada beberapa cara membuat redirect di Pyramid:

**1. Return HTTPFound (disarankan)**

```python
return HTTPFound(location='/plain')
```

**2. Raise HTTPFound**

```python
raise HTTPFound(location='/plain')
```

Keduanya memiliki efek sama, namun `raise` bisa digunakan ketika ingin menghentikan proses lebih awal di dalam fungsi yang kompleks.

### Jenis Redirect Umum

* **HTTPFound (302)** – redirect sementara
* **HTTPMovedPermanently (301)** – redirect permanen
* **HTTPSeeOther (303)** – redirect setelah POST
* **HTTPTemporaryRedirect (307)** – redirect sementara tanpa mengganti metode

---

## Extra Credit: Return vs Raise HTTPFound

### Return

```python
def home(self):
    return HTTPFound(location='/plain')
```

* Lebih mudah dibaca dan diikuti.
* Cocok untuk flow normal aplikasi.

### Raise

```python
def home(self):
    raise HTTPFound(location='/plain')
```

* Menghentikan eksekusi secara langsung.
* Berguna untuk kondisi tertentu di tengah proses logika.

**Kesimpulan:**
Keduanya menghasilkan hasil yang sama, namun `return` lebih direkomendasikan untuk kasus sederhana, sedangkan `raise` cocok digunakan di dalam fungsi helper atau kondisi kompleks.

---

## Konsep Penting

1. **Integrasi dengan WebOb**
   Pyramid menggunakan WebOb untuk menyediakan objek `Request` dan `Response` yang stabil dan konsisten.

2. **Siklus Hidup Request**
   Setiap kali pengguna mengakses URL, Pyramid membuat `Request` baru dan mengirim `Response` setelah view selesai dijalankan.

3. **Tipe Response di Pyramid**

   * `Response` → response manual.
   * `dict` → otomatis dirender ke template.
   * `HTTPException` → redirect atau error.
   * `str` → otomatis diubah jadi Response.

---

## Kesimpulan

Tutorial ini menjelaskan dasar penting dalam pembuatan aplikasi web, yaitu bagaimana cara menangani request dan response.
Dengan bantuan **WebOb** di Pyramid, kita dapat:

* Mengambil data dari pengguna dengan mudah.
* Membuat tanggapan yang sesuai dengan kebutuhan.
* Melakukan redirect dengan berbagai metode.
* Menulis dan menguji kode dengan lebih terstruktur dan dapat diandalkan.

Pyramid membuat proses ini sederhana namun tetap powerful, sehingga cocok digunakan baik untuk proyek kecil maupun aplikasi besar.
