## Three Ways of Storing and Accessing Lots of Images in Python

Dalam laporan markdown ini, akan dilakukan eksplorasi tiga metode berbeda untuk menyimpan dan mengakses sejumlah besar gambar di Python. Metode-metode tersebut meliputi:

1. **Storing Images in the Filesystem**
2. **Storing Images in a SQL Database**
3. **Storing Images in NoSQL Databases**

Setiap metode memiliki kelebihan dan kasus penggunaannya masing-masing yang akan dibahas secara detail.

### 1. Storing Images in the Filesystem/Menyimpan Gambar di Filesystem

Cara paling sederhana untuk menyimpan gambar adalah langsung di filesystem. Metode ini melibatkan menyimpan setiap gambar sebagai file di direktori.

#### Langkah-langkah:

1. **Menyimpan Gambar**: Gunakan pustaka bawaan Python untuk menyimpan gambar ke filesystem.
2. **Mengakses Gambar**: Akses gambar menggunakan path file.

#### Contoh Kode:

```python
import os
from PIL import Image

# Direktori untuk menyimpan gambar
image_dir = 'images/'

# Pastikan direktori ada
os.makedirs(image_dir, exist_ok=True)

# Menyimpan gambar
image = Image.new('RGB', (100, 100), color = 'red')
image_path = os.path.join(image_dir, 'contoh.jpg')
image.save(image_path)

# Mengakses dan membuka gambar
opened_image = Image.open(image_path)
opened_image.show()
```

#### Kelebihan:
- Sederhana dan mudah diimplementasikan.
- Akses langsung ke file gambar.

#### Kekurangan:
- Bisa lambat dengan jumlah file yang besar.
- Sulit mengelola metadata dan hubungan antar gambar.

### 2. Storing Images in a SQL Database/ Menyimpan Gambar di Basis Data SQL

Menyimpan gambar di basis data SQL melibatkan mengkonversi gambar menjadi data biner dan menyimpannya di tabel basis data.

#### Langkah-langkah:

1. **Mengkonversi Gambar ke Biner**: Gunakan pustaka Python untuk membaca gambar dan mengkonversinya ke data biner.
2. **Menyimpan Data Biner di Basis Data**: Masukkan data biner ke dalam basis data SQL.
3. **Mengambil dan Mengkonversi Kembali**: Ambil data biner dan konversikan kembali ke gambar.

#### Contoh Kode:

```python
import sqlite3
import io
from PIL import Image

# Koneksi ke basis data SQLite
conn = sqlite3.connect('images.db')
cursor = conn.cursor()

# Membuat tabel
cursor.execute('''
    CREATE TABLE IF NOT EXISTS images (
        id INTEGER PRIMARY KEY,
        name TEXT,
        image BLOB
    )
''')

# Fungsi untuk mengkonversi gambar ke biner
def convert_to_binary(image):
    with io.BytesIO() as output:
        image.save(output, format="JPEG")
        return output.getvalue()

# Menyimpan gambar ke basis data
image = Image.new('RGB', (100, 100), color = 'blue')
image_binary = convert_to_binary(image)
cursor.execute("INSERT INTO images (name, image) VALUES (?, ?)", ('contoh', image_binary))
conn.commit()

# Mengambil gambar dari basis data
cursor.execute("SELECT image FROM images WHERE name = ?", ('contoh',))
image_binary = cursor.fetchone()[0]
image = Image.open(io.BytesIO(image_binary))
image.show()

# Menutup koneksi
conn.close()
```

#### Kelebihan:
- Penyimpanan terpusat.
- Lebih mudah mengelola dan mengkueri metadata.

#### Kekurangan:
- Kompleksitas yang meningkat.
- Waktu akses yang mungkin lebih lambat karena overhead basis data.

### 3. Storing Images in NoSQL Databases/Menyimpan Gambar di Basis Data NoSQL

Basis data NoSQL, seperti MongoDB, menyediakan opsi penyimpanan yang fleksibel untuk gambar, sering kali dengan performa yang lebih baik untuk aplikasi skala besar.

#### Langkah-langkah:

1. **Mengkonversi Gambar ke Biner**: Gunakan metode yang sama untuk mengkonversi gambar ke data biner.
2. **Menyimpan Data Biner di Basis Data NoSQL**: Gunakan basis data NoSQL seperti MongoDB untuk menyimpan data biner.
3. **Mengambil dan Mengkonversi Kembali**: Ambil data biner dan konversikan kembali ke gambar.

#### Contoh Kode:

```python
from pymongo import MongoClient
import gridfs
import io
from PIL import Image

# Koneksi ke MongoDB
client = MongoClient('mongodb://localhost:27017/')
db = client['images_db']
fs = gridfs.GridFS(db)

# Menyimpan gambar ke MongoDB
image = Image.new('RGB', (100, 100), color = 'green')
with io.BytesIO() as output:
    image.save(output, format="JPEG")
    fs.put(output.getvalue(), filename='contoh.jpg')

# Mengambil gambar dari MongoDB
image_data = fs.find_one({'filename': 'contoh.jpg'}).read()
image = Image.open(io.BytesIO(image_data))
image.show()

# Menutup koneksi
client.close()
```

#### Kelebihan:
- Skalabilitas dan efisiensi untuk dataset besar.
- Skema yang fleksibel.

#### Kekurangan:
- Pengaturan yang lebih kompleks.
- Membutuhkan pustaka tambahan dan pemahaman tentang konsep NoSQL.

### Kesimpulan

Setiap metode penyimpanan dan akses gambar di Python memiliki kelebihan dan kekurangannya masing-masing:

- **Filesystem**: Sederhana dan langsung tetapi bisa menjadi sulit diatur dengan banyak file.
- **Basis Data SQL**: Terpusat dan mudah dikelola tetapi mungkin lebih lambat karena overhead basis data.
- **Basis Data NoSQL**: Skalabel dan fleksibel tetapi lebih kompleks untuk diimplementasikan.
