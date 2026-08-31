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

# Soal 3 — Mekanisme Konsensus Ethereum

## 1. Pengertian Ethereum

Ethereum adalah salah satu blockchain **Layer 1 (L1)** yang memungkinkan transaksi dan menjalankan smart contract secara terdesentralisasi. Ethereum tidak menggunakan satu server pusat, tetapi menggunakan banyak komputer atau node yang saling berkomunikasi untuk menyimpan dan memvalidasi keadaan jaringan.

Ethereum saat ini menggunakan mekanisme konsensus **Proof of Stake (PoS)**. Ethereum beralih dari Proof of Work (PoW) ke Proof of Stake pada tahun 2022. Pada PoS, proses validasi tidak lagi dilakukan oleh penambang menggunakan daya komputasi, tetapi dilakukan oleh **validator** yang melakukan staking ETH.

---

## 2. Apa Itu Proof of Stake?

Proof of Stake adalah mekanisme yang menggunakan aset yang dikunci atau di-*stake* sebagai jaminan bahwa validator akan bertindak dengan benar.

Pada Ethereum, validator melakukan staking ETH. Validator kemudian bertugas untuk:

* memeriksa apakah blok baru valid,
* memberikan suara atau **attestation** terhadap blok,
* dan pada kondisi tertentu membuat serta menyebarkan blok baru.

Jika validator menjalankan tugas dengan benar, validator dapat memperoleh reward dalam bentuk ETH. Sebaliknya, validator yang melakukan tindakan tertentu yang merugikan jaringan dapat kehilangan sebagian ETH yang di-*stake* melalui mekanisme penalti atau **slashing**.

Secara sederhana:

```text
Validator
    │
    ├── Stake ETH
    │
    ├── Memvalidasi blok
    │
    ├── Memberikan attestation
    │
    └── Mendapat reward jika berperilaku benar
```

---

## 3. Validator pada Ethereum

Validator merupakan bagian penting dalam mekanisme konsensus Ethereum.

Untuk menjadi validator secara langsung, pengguna perlu melakukan deposit **32 ETH** ke deposit contract dan menjalankan perangkat lunak yang diperlukan untuk berpartisipasi dalam jaringan. Setelah aktif, validator menerima blok dari jaringan, memeriksa transaksi di dalamnya, kemudian memberikan suara terhadap validitas blok tersebut.

Validator memiliki dua tugas utama:

1. **Mengusulkan blok (block proposal)** ketika dipilih.
2. **Memberikan attestation** atau suara terhadap blok yang dianggap valid.

Validator yang tidak menjalankan tugasnya dapat kehilangan kesempatan mendapatkan reward, sedangkan tindakan tertentu yang dianggap sebagai perilaku jahat dapat dikenai penalti atau slashing.

---

## 4. Slot dan Epoch

Untuk mengatur waktu dalam Proof of Stake, Ethereum membagi waktu menjadi **slot** dan **epoch**.

* Satu **slot** berlangsung sekitar **12 detik**.
* Satu **epoch** terdiri dari **32 slot**.

Dengan demikian, satu epoch berlangsung sekitar 6,4 menit.

Pada setiap slot, satu validator dipilih secara acak untuk menjadi **block proposer**.

Validator lainnya yang tergabung dalam committee akan memeriksa blok dan memberikan attestation. Pembagian validator ke dalam committee membantu mengurangi jumlah komunikasi yang harus dilakukan oleh setiap validator.

---

## 5. Proses Konsensus Ethereum

Secara sederhana, proses konsensus Ethereum dapat dijelaskan melalui beberapa tahap.

### Tahap 1 — Transaksi dibuat

Pengguna melakukan transaksi, misalnya mengirim ETH atau menjalankan smart contract.

Transaksi kemudian disebarkan melalui jaringan Ethereum.

### Tahap 2 — Validator dipilih sebagai block proposer

Pada setiap slot, Ethereum memilih satu validator secara acak untuk menjadi **block proposer**.

Validator tersebut mengambil transaksi yang tersedia dan membuat sebuah blok baru.

### Tahap 3 — Blok disebarkan

Blok yang dibuat oleh block proposer kemudian disebarkan ke node-node lain di jaringan.

Node lain menerima blok tersebut dan melakukan pemeriksaan.

Pemeriksaan antara lain memastikan bahwa transaksi di dalam blok valid dan perubahan state yang dihasilkan oleh transaksi tersebut benar.

### Tahap 4 — Validator memberikan Attestation

Validator yang tergabung dalam committee memberikan **attestation**.

Attestation dapat dipahami sebagai suara validator yang menyatakan bahwa validator tersebut menganggap blok tertentu valid dan merupakan bagian yang benar dari rantai blockchain.

Attestation kemudian disebarkan dan digabungkan dengan attestation dari validator lainnya.

### Tahap 5 — Menentukan rantai yang benar

Dalam kondisi normal, validator akan memiliki pandangan yang sama mengenai blok berikutnya.

Namun, dalam kondisi tertentu dapat terjadi lebih dari satu blok yang dianggap sebagai kandidat untuk posisi yang sama.

Ethereum menggunakan aturan **GHOST/LMD-GHOST** sebagai bagian dari mekanisme fork choice untuk menentukan rantai yang harus dianggap sebagai rantai utama.

Pemilihan tersebut mempertimbangkan bobot attestation dari validator, yang juga memperhitungkan jumlah ETH yang di-*stake*.

### Tahap 6 — Finality

Setelah validator memberikan suara secara cukup kuat terhadap checkpoint tertentu, jaringan dapat mencapai **finality**.

Ethereum menggunakan **Casper Friendly Finality Gadget (Casper FFG)** untuk mekanisme finalitas.

Pada setiap epoch terdapat checkpoint. Validator memberikan suara terhadap pasangan checkpoint. Jika suara tersebut mewakili setidaknya **dua pertiga (2/3)** dari total ETH yang di-*stake*, checkpoint dapat menjadi justified dan kemudian finalized melalui proses Casper FFG.

Setelah sebuah blok mencapai finality, blok tersebut secara praktis tidak dapat diubah tanpa konsekuensi ekonomi yang sangat besar bagi pihak yang mencoba menyerang jaringan.

---

## 6. Gasper

Mekanisme konsensus Ethereum secara keseluruhan dikenal sebagai **Gasper**.

Gasper merupakan gabungan dari dua komponen utama:

### a. Casper FFG

Casper FFG berfungsi untuk memberikan **finality** pada blockchain.

Dengan Casper FFG, validator memberikan suara terhadap checkpoint. Jika dukungan mencapai setidaknya dua pertiga dari total ETH yang di-*stake*, checkpoint dapat mencapai status finalized.

### b. GHOST / LMD-GHOST

GHOST digunakan sebagai **fork-choice rule**.

Fungsinya adalah membantu menentukan rantai mana yang harus dianggap sebagai rantai utama ketika terdapat beberapa kandidat blok atau terjadi percabangan.

Pemilihan dilakukan berdasarkan bobot suara validator.

Jadi secara sederhana:

```text
              GASPER
                 │
        ┌────────┴────────┐
        │                 │
    Casper FFG        GHOST/LMD-GHOST
        │                 │
    Finality          Fork Choice
        │                 │
        └────────┬────────┘
                 │
        Konsensus Ethereum
```

---

## 7. Reward dan Penalti

Proof of Stake Ethereum menggunakan sistem insentif untuk mendorong validator berperilaku jujur.

Validator yang menjalankan tugasnya dengan benar dapat memperoleh reward ETH.

Sebaliknya, validator dapat kehilangan reward jika tidak menjalankan tugasnya. Untuk tindakan tertentu yang menunjukkan perilaku jahat, seperti memberikan suara yang bertentangan atau mengusulkan blok yang tidak seharusnya, validator dapat terkena **slashing**.

Dengan mekanisme tersebut, menyerang jaringan menjadi mahal karena penyerang mempertaruhkan ETH yang dapat hilang apabila melakukan tindakan yang melanggar aturan.

---

## 8. Mengapa Proof of Stake Aman?

Keamanan Ethereum tidak hanya bergantung pada software, tetapi juga pada insentif ekonomi.

Validator memiliki ETH yang di-*stake* sebagai jaminan. Oleh karena itu, validator mempunyai kepentingan untuk mengikuti aturan jaringan.

Jika validator bertindak jujur:

```text
Stake ETH
    ↓
Validasi blok dengan benar
    ↓
Mendapat reward
```

Jika validator melakukan tindakan yang melanggar aturan:

```text
Stake ETH
    ↓
Melakukan tindakan berbahaya
    ↓
Penalti / Slashing
    ↓
ETH dapat hilang
```

Dengan cara tersebut, biaya ekonomi untuk mencoba menyerang jaringan menjadi sangat besar.

---

## 9. Ringkasan Mekanisme Konsensus Ethereum

Secara keseluruhan, mekanisme konsensus Ethereum dapat diringkas sebagai berikut:

1. Pengguna membuat transaksi.
2. Transaksi disebarkan ke jaringan.
3. Pada setiap slot, satu validator dipilih secara acak sebagai block proposer.
4. Block proposer membuat dan menyebarkan blok.
5. Validator lain memeriksa blok tersebut.
6. Validator memberikan **attestation** sebagai suara terhadap blok.
7. Fork-choice menggunakan **GHOST/LMD-GHOST** untuk menentukan rantai yang dianggap benar.
8. **Casper FFG** menggunakan suara validator terhadap checkpoint untuk mencapai finality.
9. Validator yang berperilaku benar mendapatkan reward.
10. Validator yang melakukan tindakan tertentu yang melanggar aturan dapat dikenai penalti atau slashing.

Dengan demikian, Ethereum dapat mencapai kesepakatan mengenai keadaan blockchain tanpa menggunakan penambangan berbasis komputasi seperti pada Proof of Work.

### Kesimpulan

Ethereum menggunakan **Proof of Stake** sebagai dasar mekanisme konsensusnya. Validator melakukan staking ETH untuk berpartisipasi dalam proses validasi. Pada setiap slot, validator dipilih untuk mengusulkan blok, sementara validator lainnya memberikan attestation terhadap blok tersebut.

Mekanisme konsensus Ethereum secara keseluruhan disebut **Gasper**, yang menggabungkan **Casper FFG** untuk finality dan **GHOST/LMD-GHOST** untuk fork choice. Kombinasi tersebut memungkinkan jaringan Ethereum menentukan rantai yang benar, mencapai finality, serta memberikan insentif ekonomi agar validator berperilaku jujur.




