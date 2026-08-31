# Responsi Praktikum Sistem Terdistribusi dan Terdesentralisasi

## Soal 1 — Menjalankan YugabyteDB dan Membuat Tabel

### Tujuan

Menjalankan YugabyteDB menggunakan Docker, kemudian membuat dua tabel dan mengisi masing-masing tabel dengan lima data.

### 1. Menjalankan YugabyteDB

YugabyteDB dijalankan menggunakan Docker dengan perintah:

```bash
docker run -d --name yugabytedb -p 7000:7000 -p 9000:9000 -p 15433:15433 -p 5433:5433 -p 9042:9042 yugabytedb/yugabyte:latest bin/yugabyted start --daemon=false
```

Setelah container dijalankan, dilakukan pengecekan menggunakan:

```bash
docker ps
```

Container YugabyteDB berhasil berjalan dengan status `Up`.

### 2. Mengakses YSQL

YSQL diakses melalui container menggunakan:

```bash
docker exec -it yugabytedb ysqlsh -h c2798db4ad87 -p 5433
```

Setelah berhasil terhubung, muncul prompt:

```text
yugabyte=#
```

### 3. Membuat Tabel `mahasiswa`

Tabel pertama dibuat dengan perintah:

```sql
CREATE TABLE mahasiswa (
    id SERIAL PRIMARY KEY,
    nama VARCHAR(100) NOT NULL,
    jurusan VARCHAR(100) NOT NULL
);
```

Hasil:

```text
CREATE TABLE
```

### 4. Membuat Tabel `nilai`

Tabel kedua dibuat dengan perintah:

```sql
CREATE TABLE nilai (
    id SERIAL PRIMARY KEY,
    mahasiswa_id INT NOT NULL,
    mata_kuliah VARCHAR(100) NOT NULL,
    nilai INT NOT NULL
);
```

Hasil:

```text
CREATE TABLE
```

### 5. Mengisi Data Tabel `mahasiswa`

Lima data dimasukkan menggunakan perintah:

```sql
INSERT INTO mahasiswa (nama, jurusan) VALUES
('Andi', 'Informatika'),
('Budi', 'Informatika'),
('Citra', 'Sistem Informasi'),
('Deni', 'Informatika'),
('Eka', 'Sistem Informasi');
```

Hasil:

```text
INSERT 0 5
```

### 6. Mengisi Data Tabel `nilai`

Lima data dimasukkan menggunakan perintah:

```sql
INSERT INTO nilai (mahasiswa_id, mata_kuliah, nilai) VALUES
(1, 'Basis Data', 85),
(2, 'Sistem Terdistribusi', 90),
(3, 'Pemrograman Web', 88),
(4, 'Jaringan Komputer', 82),
(5, 'Basis Data', 91);
```

Hasil:

```text
INSERT 0 5
```

### 7. Membuktikan Dua Tabel Berhasil Dibuat

Daftar tabel diperiksa menggunakan:

```sql
\dt
```

Hasil:

```text
           List of relations
 Schema |   Name    | Type  |  Owner
--------+-----------+-------+----------
 public | mahasiswa | table | yugabyte
 public | nilai     | table | yugabyte
(2 rows)
```

Hasil tersebut menunjukkan bahwa terdapat dua tabel, yaitu:

* `mahasiswa`
* `nilai`

### 8. Membuktikan Data Berhasil Dimasukkan

Data pada tabel `mahasiswa` diperiksa menggunakan:

```sql
SELECT * FROM mahasiswa;
```

Hasil menunjukkan terdapat **5 data mahasiswa**.

Data pada tabel `nilai` diperiksa menggunakan:

```sql
SELECT * FROM nilai;
```

Hasil menunjukkan terdapat **5 data nilai**.

### Kesimpulan Soal 1

YugabyteDB berhasil dijalankan menggunakan Docker. Dua tabel, yaitu `mahasiswa` dan `nilai`, berhasil dibuat dan masing-masing tabel telah diisi dengan lima data.

---

# Soal 2 — REST API Menggunakan Python

### Tujuan

Membuat REST API menggunakan Python untuk mengekspos data yang telah dibuat pada Soal 1.

REST API dibuat menggunakan **FastAPI**, sedangkan koneksi ke YugabyteDB menggunakan **Psycopg**.

### 1. Membuat Virtual Environment

Virtual environment dibuat menggunakan Python 3.14.4:

```bash
uv venv --python 3.14.4
```

Kemudian environment diaktifkan:

```bash
source .venv/bin/activate
```

### 2. Instalasi Library

Library yang digunakan diinstal dengan perintah:

```bash
uv pip install fastapi uvicorn psycopg[binary]
```

Library yang digunakan:

* `FastAPI` — framework untuk membuat REST API.
* `Uvicorn` — server untuk menjalankan aplikasi FastAPI.
* `Psycopg` — library Python untuk koneksi ke YugabyteDB melalui PostgreSQL.

### 3. Membuat REST API

REST API dibuat dalam file `app.py`.

```python
from fastapi import FastAPI
import psycopg

app = FastAPI(title="REST API YugabyteDB")

DB_CONFIG = {
    "host": "127.0.0.1",
    "port": 5433,
    "dbname": "yugabyte",
    "user": "yugabyte",
    "password": "yugabyte",
}


def get_connection():
    return psycopg.connect(**DB_CONFIG)


@app.get("/")
def root():
    return {
        "message": "REST API YugabyteDB berhasil berjalan"
    }


@app.get("/mahasiswa")
def get_mahasiswa():
    conn = get_connection()

    with conn.cursor() as cur:
        cur.execute(
            "SELECT id, nama, jurusan FROM mahasiswa ORDER BY id"
        )
        rows = cur.fetchall()

    conn.close()

    return [
        {
            "id": row[0],
            "nama": row[1],
            "jurusan": row[2]
        }
        for row in rows
    ]


@app.get("/nilai")
def get_nilai():
    conn = get_connection()

    with conn.cursor() as cur:
        cur.execute(
            """
            SELECT id, mahasiswa_id, mata_kuliah, nilai
            FROM nilai
            ORDER BY id
            """
        )
        rows = cur.fetchall()

    conn.close()

    return [
        {
            "id": row[0],
            "mahasiswa_id": row[1],
            "mata_kuliah": row[2],
            "nilai": row[3]
        }
        for row in rows
    ]
```

### 4. Menjalankan REST API

Server dijalankan menggunakan Uvicorn:

```bash
uvicorn app:app --host 0.0.0.0 --port 8000
```

Server berhasil berjalan pada:

```text
http://0.0.0.0:8000
```

### 5. Pengujian Endpoint Utama

Pengujian dilakukan menggunakan `curl`:

```bash
curl http://127.0.0.1:8000/
```

Hasil:

```json
{"message":"REST API YugabyteDB berhasil berjalan"}
```

Hasil tersebut menunjukkan bahwa REST API berhasil berjalan.

### 6. Pengujian Endpoint `/mahasiswa`

Perintah:

```bash
curl http://127.0.0.1:8000/mahasiswa
```

Hasil:

```json
[{"id":1,"nama":"Andi","jurusan":"Informatika"},{"id":2,"nama":"Budi","jurusan":"Informatika"},{"id":3,"nama":"Citra","jurusan":"Sistem Informasi"},{"id":4,"nama":"Deni","jurusan":"Informatika"},{"id":5,"nama":"Eka","jurusan":"Sistem Informasi"}]
```

Endpoint `/mahasiswa` berhasil mengambil lima data dari tabel `mahasiswa` dan mengembalikannya dalam format JSON.

### 7. Pengujian Endpoint `/nilai`

Perintah:

```bash
curl http://127.0.0.1:8000/nilai
```

Hasil:

```json
[{"id":1,"mahasiswa_id":1,"mata_kuliah":"Basis Data","nilai":85},{"id":2,"mahasiswa_id":2,"mata_kuliah":"Sistem Terdistribusi","nilai":90},{"id":3,"mahasiswa_id":3,"mata_kuliah":"Pemrograman Web","nilai":88},{"id":4,"mahasiswa_id":4,"mata_kuliah":"Jaringan Komputer","nilai":82},{"id":5,"mahasiswa_id":5,"mata_kuliah":"Basis Data","nilai":91}]
```

Endpoint `/nilai` berhasil mengambil lima data dari tabel `nilai` dan mengembalikannya dalam format JSON.

### Kesimpulan Soal 2

REST API menggunakan Python dan FastAPI berhasil dibuat. API dapat mengambil data dari YugabyteDB dan mengekspos data melalui endpoint:

* `GET /`
* `GET /mahasiswa`
* `GET /nilai`

Pengujian menggunakan `curl` menunjukkan bahwa data berhasil ditampilkan dalam format JSON.

---

# Bukti Dokumentasi

Screenshot yang dapat disertakan dalam repository:

1. Container YugabyteDB yang sedang berjalan menggunakan Docker.
2. Daftar tabel menggunakan `\dt`.
3. Isi tabel `mahasiswa`.
4. Isi tabel `nilai`.
5. FastAPI yang sedang berjalan menggunakan Uvicorn.
6. Hasil `curl` endpoint `/mahasiswa`.
7. Hasil `curl` endpoint `/nilai`.
