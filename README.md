[![en](https://img.shields.io/badge/lang-en-red.svg)](https://github.com/mukhtarulll/archaeological-database/blob/main/README.en.md)

# Sistem Database Manajemen Penelitian Arkeologi
## Project: Database Design & Dashboard Visualization

Proyek kelompok ini merancang dan mengimplementasikan basis data relasional ternormalisasi (MySQL) untuk mengelola data penelitian arkeologi, termasuk ekspedisi, penemuan artefak, lokasi situs, dan hasil laboratorium. Dashboard interaktif dibangun menggunakan Budibase untuk visualisasi real-time aktivitas penelitian dan analisis artefak.

---

## Ringkasan Singkat

![Dashboard Arkeologi](/Dokumentasi%20BudiBase/Dashboard%20Awal.png)

Dashboard menampilkan:
- **Total 11 Artefak** ditemukan dari **5 Arkeolog** di **9 Situs**
- **Distribusi Artefak Berdasarkan Budaya** — Hindu-Buddha (18.2%), Sound Horeg (18.2%), Asdn (18.2%), dll
- **Top 5 Situs Paling Aktif** — Situs Gunung Padang memimpin dengan 12 ekspedisi
- **Aktivitas Ekspedisi per Bulan** — Tren ekspedisi Juli-Sept 2023 dan Juni 2025
- **Data Hasil Lab Terbaru** — Analisis morfologi, komposisi, dan dating artefak

---

## Arsitektur Database

### Entity-Relationship Diagram (ERD)
Database menggunakan **star schema** ternormalisasi dengan 7 entitas utama:

![ERD](ERD%20Arkeolog.drawio.jpg)

### Tabel Utama

| Tabel | Primary Key | Description | Relationships |
|-------|-------------|-------------|---------------|
| `arkeolog` | id_arkeolog | Data peneliti (nama, institusi, email) | 1:N → ekspedisi |
| `situs_arkeologi` | id_situs | Data situs (nama, lokasi geografis, periode) | 1:N → lokasi |
| `lokasi` | id_lokasi | Grid ekskavasi spesifik di situs | N:1 ← situs, 1:N → artefak |
| `ekspedisi` | id_ekspedisi | Kegiatan penelitian (tanggal, durasi, arkeolog) | N:1 ← arkeolog, M:N ↔ artefak |
| `artefak` | id_artefak | Temuan arkeologi (jenis, dimensi, kondisi) | N:1 ← lokasi, M:N ↔ ekspedisi |
| `periode_budaya` | id_periode | Klasifikasi temporal (Hindu-Buddha, Megalitik, dll) | 1:N → artefak |
| `hasil_laboratorium` | id_hasil | Hasil analisis lab (dating, komposisi, morfologi) | N:1 ← artefak |

---

## Fitur Database

### 1. Normalisasi (3NF)
- **Eliminasi redundansi data** dengan foreign key relationships
- **Atomic values** di semua kolom
- **No transitive dependencies** — setiap non-key attribute hanya bergantung pada primary key

### 2. Indexing
```sql
-- Index untuk query performa
CREATE INDEX idx_situs_nama ON situs_arkeologi(nama_situs);
CREATE INDEX idx_artefak_periode ON artefak(id_periode);
CREATE INDEX idx_ekspedisi_tanggal ON ekspedisi(tanggal_mulai);
```

### 3. Referential Integrity
- Foreign key constraints untuk menjaga konsistensi data
- CASCADE delete untuk junction table (artifact_ekspedisi)
- RESTRICT delete untuk master data (situs, arkeolog)

---

## Query Analitik

### Top 5 Situs Paling Produktif
```sql
SELECT 
    s.nama_situs,
    s.lokasi_geografis,
    COUNT(DISTINCT e.id_ekspedisi) as jumlah_ekspedisi,
    COUNT(DISTINCT a.id_artefak) as total_artefak
FROM situs_arkeologi s
LEFT JOIN lokasi l ON s.id_situs = l.id_situs
LEFT JOIN artefak a ON l.id_lokasi = a.id_lokasi
LEFT JOIN artefak_ekspedisi ae ON a.id_artefak = ae.id_artefak
LEFT JOIN ekspedisi e ON ae.id_ekspedisi = e.id_ekspedisi
GROUP BY s.id_situs, s.nama_situs, s.lokasi_geografis
ORDER BY jumlah_ekspedisi DESC, total_artefak DESC
LIMIT 5;
```

![Top 5 Situs](/Dokumentasi%20BudiBase/Top%205%20Situs%20AKtif.png)

### Distribusi Artefak Berdasarkan Periode Budaya
```sql
SELECT 
    pb.nama_periode,
    pb.rentang_tahun,
    COUNT(*) as jumlah_artefak,
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM artefak), 1) as persentase
FROM periode_budaya pb
LEFT JOIN artefak a ON pb.id_periode = a.id_periode
GROUP BY pb.id_periode, pb.nama_periode, pb.rentang_tahun
ORDER BY jumlah_artefak DESC;
```

### Aktivitas Ekspedisi per Bulan
```sql
SELECT 
    DATE_FORMAT(tanggal_mulai, '%Y-%m') as bulan,
    COUNT(*) as jumlah_ekspedisi
FROM ekspedisi
GROUP BY DATE_FORMAT(tanggal_mulai, '%Y-%m')
ORDER BY bulan;
```

![Aktivitas Ekspedisi](/Dokumentasi%20BudiBase/Aktifitas%20Ekspedisi%20per%20Bulan.png)

### Arkeolog Terprodukitf
```sql
SELECT 
    ar.nama,
    ar.institusi,
    COUNT(e.id_ekspedisi) as jumlah_ekspedisi,
    COUNT(DISTINCT ae.id_artefak) as artefak_ditemukan
FROM arkeolog ar
LEFT JOIN ekspedisi e ON ar.id_arkeolog = e.id_arkeolog
LEFT JOIN artefak_ekspedisi ae ON e.id_ekspedisi = ae.id_ekspedisi
GROUP BY ar.id_arkeolog, ar.nama, ar.institusi
ORDER BY jumlah_ekspedisi DESC, artefak_ditemukan DESC;
```

![Akrkeolog Teproduktif](/Dokumentasi%20BudiBase/Arkeolog%20Terproduktif.png)

Lihat file `queries.sql` untuk query lengkap dan kompleks lainnya.

---

## Dashboard Budibase

### Komponen Dashboard
1. **KPI Cards**
   - Total Artefak: 11
   - Total Arkeolog: 5 orang
   - Total Situs: 9 lokasi

2. **Pie Chart: Distribusi Artefak Berdasarkan Budaya**
   - Visual proporsi artefak per periode (Hindu-Buddha, Sound Horeg, Megalitik, dll)

3. **Bar Chart: Aktivitas Ekspedisi per Bulan**
   - Tren temporal ekspedisi penelitian (spikes di July-Sept 2023 & June 2025)

4. **Horizontal Bar: Top 5 Situs Paling Aktif**
   - Ranking situs berdasarkan jumlah ekspedisi

5. **Data Table: Hasil Lab Terbaru**
   - List analisis lab terkini dengan tanggal, artefak, hasil

6. **Gallery: Dokumentasi Artefak**
   - Foto-foto temuan arkeologi dari ekskavasi

### Koneksi Database
- **Backend**: MySQL 10.4 (MariaDB)
- **Frontend**: Budibase (low-code dashboard builder)
- **Connection**: Direct MySQL connection dengan credentials read-only untuk dashboard

*Note: Server database sudah tidak aktif (server dosen dimatikan), namun schema dan queries masih tersedia untuk reproducibility.*

---

## Cara Setup (Lokal)

### Prerequisites
- MySQL 8.0+ atau MariaDB 10.4+
- Budibase (Docker atau cloud instance)
- phpMyAdmin (opsional, untuk GUI)

### Installation

#### 1. Setup Database
```bash
# Login ke MySQL
mysql -u root -p

# Import database
mysql -u root -p < arkeologidb.sql

# Verify import
mysql -u root -p arkeologidb
SHOW TABLES;
```

#### 2. Test Queries
```bash
# Jalankan sample queries
mysql -u root -p arkeologidb < queries.sql
```

---

## Tech Stack

| Component | Technology |
|-----------|------------|
| **Database** | MySQL 10.4 (MariaDB) |
| **Normalization** | Third Normal Form (3NF) |
| **Dashboard** | Budibase (Low-code platform) |
| **Query Language** | SQL (SELECT, JOIN, GROUP BY, aggregations) |
| **Documentation** | ERD (Crow's foot notation), Data Dictionary |
| **Version Control** | Git, GitHub |

---

## Sample Data

Database terisi dengan data simulasi:
- **5 Arkeolog** dari berbagai institusi (UNNES, UI, UGM)
- **9 Situs Arkeologi** di Indonesia (Borobudur, Gunung Padang, Trowulan, dll)
- **11 Artefak** dengan klasifikasi periode budaya
- **8 Ekspedisi** penelitian (Juli-Sept 2023, Juni 2025)
- **6 Hasil Laboratorium** (analisis morfologi, komposisi, dating)

---

## Limitasi & Future Work

### Limitasi
1. **No authentication/authorization** — Database belum ada user management untuk akses control
2. **Static dashboard** — Budibase dashboard tidak auto-refresh (perlu manual reload)
3. **Limited sample data** — Hanya 11 artefak untuk demonstrasi, real-world akan ribuan record
4. **No GIS integration** — Lokasi geografis masih text field, belum ada map visualization

### Future Work
1. **Add user roles** — Implement RBAC (arkeolog, lab analyst, public viewer)
2. **GIS integration** — Integrasikan Google Maps/Leaflet untuk visualisasi situs
3. **Image storage** — Tambah tabel foto_artefak dengan blob storage atau S3
4. **Time-series analysis** — Track perubahan kondisi artefak over time (conservation monitoring)
5. **Export reports** — Generate PDF laporan ekspedisi otomatis dari database

---

## Lisensi

- **Database Schema**: Open untuk educational purposes
- **Sample Data**: Simulasi untuk academic project
- **Budibase**: Open-source (GPL v3)
